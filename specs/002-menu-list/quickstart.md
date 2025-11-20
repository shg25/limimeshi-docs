# Quickstart: メニュー一覧（Menu List）

**Feature**: 002-menu-list
**Date**: 2025-11-18
**Related**: [spec.md](./spec.md), [research.md](./research.md), [contracts/firestore-schema.md](./contracts/firestore-schema.md)

## Prerequisites

- Node.js 18+ インストール済み
- Firebase Project作成済み（`limimeshi-dev` or `limimeshi-prod`）
- Firestore有効化済み
- Firebase Authentication有効化済み（Googleログイン、匿名認証）
- limimeshi-web リポジトリがセットアップ済み

---

## Setup

### 1. 依存関係のインストール

```bash
cd limimeshi-web
npm install
```

**主要な依存関係**:
- `react` `react-dom`: フロントエンドフレームワーク
- `firebase`: Firebase SDK
- `@mui/material @emotion/react @emotion/styled`: Material-UI
- `react-twitter-embed`: X Post埋め込み
- `typescript`: 型安全性
- `vite`: ビルドツール
- `vitest @testing-library/react`: テスト

### 2. Firebase設定

`src/firebase/config.ts`:
```typescript
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
export const auth = getAuth(app);
```

`.env.local`:
```
VITE_FIREBASE_API_KEY=your-api-key
VITE_FIREBASE_AUTH_DOMAIN=limimeshi-dev.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=limimeshi-dev
VITE_FIREBASE_STORAGE_BUCKET=limimeshi-dev.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abcdef
```

### 3. Firestoreインデックスの作成

Firebase Consoleで以下のインデックスを作成:

#### Index 1: saleStartTime（降順）
- Collection: `menus`
- Fields: `saleStartTime` (Descending)

#### Index 2: chainId + saleStartTime
- Collection: `menus`
- Fields:
  - `chainId` (Ascending)
  - `saleStartTime` (Descending)

または、`firestore.indexes.json`（limimeshi-infraリポジトリ、Phase3以降）:
```json
{
  "indexes": [
    {
      "collectionGroup": "menus",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "saleStartTime", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "menus",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "chainId", "order": "ASCENDING" },
        { "fieldPath": "saleStartTime", "order": "DESCENDING" }
      ]
    }
  ]
}
```

---

## Core Implementation

### 1. メニュー一覧コンポーネント

`src/pages/MenuList.tsx`:
```typescript
import { useState, useEffect } from 'react';
import { collection, query, where, orderBy, getDocs, Timestamp } from 'firebase/firestore';
import { db, auth } from '../firebase/config';
import { Container, Grid, Card, CardContent, Typography, Switch, FormControlLabel } from '@mui/material';
import { Menu, Chain, MenuWithChain } from '../types';
import { getMenuStatus } from '../utils/menuStatus';
import { TwitterTweetEmbed } from 'react-twitter-embed';

export const MenuListPage = () => {
  const [menus, setMenus] = useState<MenuWithChain[]>([]);
  const [loading, setLoading] = useState(true);
  const [showFavoritesOnly, setShowFavoritesOnly] = useState(() => {
    // localStorageからフィルタ設定を読み込み
    const saved = localStorage.getItem('menuFilter');
    return saved ? JSON.parse(saved).showFavoritesOnly : false;
  });
  const [user, setUser] = useState(auth.currentUser);

  // 認証状態の監視
  useEffect(() => {
    const unsubscribe = auth.onAuthStateChanged(setUser);
    return unsubscribe;
  }, []);

  // メニュー一覧の取得
  useEffect(() => {
    const fetchMenus = async () => {
      setLoading(true);

      try {
        // 1年前の日時を計算
        const oneYearAgo = Timestamp.fromDate(
          new Date(Date.now() - 365 * 24 * 60 * 60 * 1000)
        );

        let menusQuery;

        if (showFavoritesOnly && user) {
          // お気に入りチェーンID一覧を取得
          const favoritesSnapshot = await getDocs(
            collection(db, 'users', user.uid, 'favorites')
          );
          const favoriteChainIds = favoritesSnapshot.docs.map(doc => doc.id);

          if (favoriteChainIds.length === 0) {
            setMenus([]);
            setLoading(false);
            return;
          }

          // お気に入りチェーンのメニューのみ取得
          menusQuery = query(
            collection(db, 'menus'),
            where('chainId', 'in', favoriteChainIds.slice(0, 10)), // 最大10個
            where('saleStartTime', '>=', oneYearAgo),
            orderBy('saleStartTime', 'desc')
          );
        } else {
          // 全メニュー取得
          menusQuery = query(
            collection(db, 'menus'),
            where('saleStartTime', '>=', oneYearAgo),
            orderBy('saleStartTime', 'desc')
          );
        }

        const snapshot = await getDocs(menusQuery);

        // チェーン名を取得してマージ
        const chainsCache = new Map<string, string>();
        const menusWithChain: MenuWithChain[] = await Promise.all(
          snapshot.docs.map(async (doc) => {
            const menu = { id: doc.id, ...doc.data() } as Menu;

            // チェーン名をキャッシュから取得、なければFirestoreから取得
            let chainName = chainsCache.get(menu.chainId);
            if (!chainName) {
              const chainDoc = await getDocs(
                query(collection(db, 'chains'), where('__name__', '==', menu.chainId))
              );
              chainName = chainDoc.docs[0]?.data()?.name || '不明';
              chainsCache.set(menu.chainId, chainName);
            }

            return {
              ...menu,
              chainName,
              status: getMenuStatus(menu.saleStartTime, menu.saleEndTime),
            };
          })
        );

        setMenus(menusWithChain);
      } catch (error) {
        console.error('Error fetching menus:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchMenus();
  }, [showFavoritesOnly, user]);

  // フィルタ切り替え時にlocalStorageに保存
  const handleFilterChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = event.target.checked;
    setShowFavoritesOnly(newValue);
    localStorage.setItem('menuFilter', JSON.stringify({ showFavoritesOnly: newValue }));
  };

  // X Post IDを抽出
  const extractTweetId = (url?: string): string | null => {
    if (!url) return null;
    const match = url.match(/status\/(\d+)/);
    return match ? match[1] : null;
  };

  if (loading) {
    return <Typography>Loading...</Typography>;
  }

  return (
    <Container>
      <Typography variant="h4" gutterBottom>
        期間限定メニュー一覧
      </Typography>

      {user && (
        <FormControlLabel
          control={
            <Switch
              checked={showFavoritesOnly}
              onChange={handleFilterChange}
            />
          }
          label="お気に入りのみ表示"
        />
      )}

      {menus.length === 0 ? (
        <Typography>データなし</Typography>
      ) : (
        <Grid container spacing={2}>
          {menus.map((menu) => {
            const tweetId = extractTweetId(menu.xPostUrl);

            return (
              <Grid item xs={12} md={6} key={menu.id}>
                <Card>
                  <CardContent>
                    <Typography variant="h6">{menu.name}</Typography>
                    <Typography variant="body2" color="text.secondary">
                      {menu.chainName}
                    </Typography>
                    <Typography variant="body2">{menu.status}</Typography>
                    <Typography variant="body2">
                      販売開始: {menu.saleStartTime.toDate().toLocaleDateString()}
                    </Typography>
                    {menu.saleEndTime && (
                      <Typography variant="body2">
                        販売終了: {menu.saleEndTime.toDate().toLocaleDateString()}
                      </Typography>
                    )}
                    {menu.description && (
                      <Typography variant="body2" sx={{ mt: 1 }}>
                        {menu.description}
                      </Typography>
                    )}
                    {tweetId && (
                      <TwitterTweetEmbed tweetId={tweetId} />
                    )}
                  </CardContent>
                </Card>
              </Grid>
            );
          })}
        </Grid>
      )}
    </Container>
  );
};
```

### 2. ステータス判定ユーティリティ

`src/utils/menuStatus.ts`:
```typescript
import { Timestamp } from 'firebase/firestore';

export type MenuStatus = '予定' | string; // '発売から◯日経過' など

export const getMenuStatus = (
  saleStartTime: Timestamp,
  saleEndTime?: Timestamp
): MenuStatus => {
  const now = new Date();
  const startDate = saleStartTime.toDate();
  const endDate = saleEndTime?.toDate();

  // 販売終了日時が設定されていて、現在時刻が終了日時を過ぎている
  if (endDate && now > endDate) {
    return '販売終了';
  }

  // 販売開始日時が未来
  if (startDate > now) {
    return '予定';
  }

  // 販売開始日時から経過日数を計算
  const daysPassed = Math.floor(
    (now.getTime() - startDate.getTime()) / (1000 * 60 * 60 * 24)
  );

  // 30日以内
  if (daysPassed < 30) {
    return `発売から${daysPassed}日経過`;
  }

  // 30日以上
  const monthsPassed = Math.floor(daysPassed / 30);
  return `発売から${monthsPassed}ヶ月以上経過`;
};
```

### 3. 型定義

`src/types/index.ts`:
```typescript
import { Timestamp } from 'firebase/firestore';

export interface Menu {
  id: string;
  chainId: string;
  name: string;
  description?: string;
  xPostUrl?: string;
  saleStartTime: Timestamp;
  saleEndTime?: Timestamp;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

export interface Chain {
  id: string;
  name: string;
  furigana: string;
  officialUrl?: string;
  logoUrl?: string;
  favoriteCount: number;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

export type MenuStatus = '予定' | '発売から◯日経過' | '発売から◯ヶ月以上経過' | '販売終了';

export interface MenuWithChain extends Menu {
  chainName: string;
  status: MenuStatus;
}
```

---

## Testing

### 単体テスト（ステータス判定ロジック）

`src/utils/menuStatus.test.ts`:
```typescript
import { describe, it, expect } from 'vitest';
import { Timestamp } from 'firebase/firestore';
import { getMenuStatus } from './menuStatus';

describe('getMenuStatus', () => {
  it('販売終了日時を過ぎている場合、「販売終了」を返す', () => {
    const saleStartTime = Timestamp.fromDate(new Date('2025-01-01'));
    const saleEndTime = Timestamp.fromDate(new Date('2025-01-31'));
    expect(getMenuStatus(saleStartTime, saleEndTime)).toBe('販売終了');
  });

  it('販売開始日時が未来の場合、「予定」を返す', () => {
    const saleStartTime = Timestamp.fromDate(new Date('2026-01-01'));
    expect(getMenuStatus(saleStartTime)).toBe('予定');
  });

  it('販売開始から10日経過している場合、「発売から10日経過」を返す', () => {
    const tenDaysAgo = new Date();
    tenDaysAgo.setDate(tenDaysAgo.getDate() - 10);
    const saleStartTime = Timestamp.fromDate(tenDaysAgo);
    expect(getMenuStatus(saleStartTime)).toBe('発売から10日経過');
  });

  it('販売開始から60日経過している場合、「発売から2ヶ月以上経過」を返す', () => {
    const sixtyDaysAgo = new Date();
    sixtyDaysAgo.setDate(sixtyDaysAgo.getDate() - 60);
    const saleStartTime = Timestamp.fromDate(sixtyDaysAgo);
    expect(getMenuStatus(saleStartTime)).toBe('発売から2ヶ月以上経過');
  });
});
```

### E2Eテスト

`tests/e2e/menu-list.spec.ts`:
```typescript
import { test, expect } from '@playwright/test';

test.describe('メニュー一覧', () => {
  test('メニューが販売開始日時降順で表示される', async ({ page }) => {
    await page.goto('/');
    await expect(page.locator('h4')).toHaveText('期間限定メニュー一覧');

    // 最初のメニューカードが表示されることを確認
    const firstMenu = page.locator('[data-testid="menu-card"]').first();
    await expect(firstMenu).toBeVisible();
  });

  test('お気に入りフィルタが機能する', async ({ page }) => {
    // ログイン済みユーザーとして操作
    await page.goto('/');
    await page.locator('[data-testid="favorites-filter"]').click();

    // お気に入りチェーンのメニューのみ表示されることを確認
    await expect(page.locator('[data-testid="menu-card"]')).toHaveCount(3);
  });
});
```

---

## Deployment

### 開発環境

```bash
# Firestore Emulatorで動作確認
firebase emulators:start

# 開発サーバー起動
npm run dev
```

### 本番環境

```bash
# ビルド
npm run build

# Firebase Hostingにデプロイ
firebase use prod
firebase deploy --only hosting:web
```

---

## Troubleshooting

### Q1: Firestoreクエリが遅い
- **A**: インデックスが作成されているか確認。Firebase Consoleでインデックスステータスを確認。

### Q2: X Post埋め込みが表示されない
- **A**: Post IDが正しく抽出されているか確認。Post作成者が削除した可能性もあり。

### Q3: お気に入りフィルタが動作しない
- **A**: ログイン済みユーザーか確認。`auth.currentUser` が null でないか確認。

### Q4: フィルタ設定が保存されない
- **A**: localStorageが有効か確認。プライベートブラウジングモードでは無効化される。

---

## Next Steps

1. **003-favorites機能の実装**: お気に入り登録・解除機能
2. **パフォーマンス最適化**: メモ化、仮想スクロール検討
3. **アクセシビリティ改善**: スクリーンリーダー対応、キーボード操作
4. **エラーハンドリング**: ネットワークエラー時の再試行ロジック
