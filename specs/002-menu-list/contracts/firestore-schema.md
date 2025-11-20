# Firestore Schema: メニュー一覧（Menu List）

**Feature**: 002-menu-list  
**Date**: 2025-11-18  
**Status**: Phase 1 - Contract  
**Related**: [research.md](../research.md), [001-admin-panel Firestore Schema](../../001-admin-panel/contracts/firestore-schema.md)  

## Overview

メニュー一覧機能で使用するFirestoreコレクション構造（読み取り専用）

## Collections（読み取り専用）

### /menus/{menuId}

期間限定メニュー（001-admin-panelで登録されたデータを読み取り）

```typescript
{
  // Firestore自動生成ID（ドキュメントID）
  id: string;

  // 関連
  chainId: string;               // 所属チェーンID（/chains/{chainId}への参照）

  // 基本情報
  name: string;                  // メニュー名
  description?: string;          // 説明

  // 外部リンク
  xPostUrl?: string;             // X Post URL

  // 販売期間
  saleStartTime: Timestamp;      // 販売開始日時
  saleEndTime?: Timestamp;       // 販売終了日時

  // メタデータ
  createdAt: Timestamp;          // 作成日時（Firebase Server Timestamp）
  updatedAt: Timestamp;          // 更新日時（Firebase Server Timestamp）
}
```

**Field Details**:

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `chainId` | string | Yes | - | /chains/{chainId}への参照 |
| `name` | string | Yes | - | 最大200文字 |
| `description` | string | No | null | 最大1000文字 |
| `xPostUrl` | string | No | null | X Post URL形式 |
| `saleStartTime` | Timestamp | Yes | - | 販売開始日時 |
| `saleEndTime` | Timestamp | No | null | 販売終了日時（未定の場合はnull） |

**Queries Used**:

1. **全メニュー取得（1年以内、販売開始日時降順）**:
   ```typescript
   const oneYearAgo = Timestamp.fromDate(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000));

   query(
     collection(db, 'menus'),
     where('saleStartTime', '>=', oneYearAgo),
     orderBy('saleStartTime', 'desc')
   );
   ```

2. **お気に入りフィルタ（chainIdの配列でフィルタ）**:
   ```typescript
   query(
     collection(db, 'menus'),
     where('chainId', 'in', favoriteChainIds),  // 最大10個
     where('saleStartTime', '>=', oneYearAgo),
     orderBy('saleStartTime', 'desc')
   );
   ```

**Indexes Required**:
- Single field index: `saleStartTime` (descending)
- Composite index: `chainId` (ascending) + `saleStartTime` (descending)

**Example Document**:
```json
{
  "chainId": "abc123",
  "name": "月見バーガー",
  "description": "ふんわりたまごとジューシーなビーフパティ",
  "xPostUrl": "https://twitter.com/McDonaldsJapan/status/1234567890",
  "saleStartTime": "2025-09-01T00:00:00.000Z",
  "saleEndTime": "2025-09-30T23:59:59.999Z",
  "createdAt": "2025-08-15T10:00:00.000Z",
  "updatedAt": "2025-08-15T10:00:00.000Z"
}
```

---

### /chains/{chainId}

チェーン店マスタ（001-admin-panelで登録されたデータを読み取り）

```typescript
{
  // Firestore自動生成ID（ドキュメントID）
  id: string;

  // 基本情報
  name: string;                  // チェーン名
  furigana: string;              // ふりがな（50音順ソート用）

  // 追加情報
  officialUrl?: string;          // 公式サイトURL
  logoUrl?: string;              // ロゴ画像URL

  // 集約データ
  favoriteCount: number;         // お気に入り登録数（デフォルト: 0）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（Firebase Server Timestamp）
  updatedAt: Timestamp;          // 更新日時（Firebase Server Timestamp）
}
```

**Queries Used**:

1. **チェーン名取得（メニューのchainIdから）**:
   ```typescript
   const chainDoc = await getDoc(doc(db, 'chains', menu.chainId));
   const chainName = chainDoc.data()?.name;
   ```

**Example Document**:
```json
{
  "name": "マクドナルド",
  "furigana": "まくどなるど",
  "officialUrl": "https://www.mcdonalds.co.jp/",
  "logoUrl": "https://example.com/logos/mcdonalds.png",
  "favoriteCount": 0,
  "createdAt": "2025-11-16T10:00:00.000Z",
  "updatedAt": "2025-11-16T10:00:00.000Z"
}
```

---

### /users/{userId}/favorites/{chainId}

お気に入りチェーン（003-favoritesで登録されたデータを読み取り）

```typescript
{
  // Firestore自動生成ID（ドキュメントID） = chainId
  chainId: string;               // お気に入り登録したチェーンID

  // メタデータ
  createdAt: Timestamp;          // 登録日時（Firebase Server Timestamp）
}
```

**Queries Used**:

1. **ログインユーザーのお気に入りチェーンID一覧**:
   ```typescript
   const favoritesSnapshot = await getDocs(
     collection(db, 'users', userId, 'favorites')
   );

   const favoriteChainIds = favoritesSnapshot.docs.map(doc => doc.id);
   ```

**Example Document**:
```json
{
  "chainId": "abc123",
  "createdAt": "2025-11-18T10:00:00.000Z"
}
```

---

## Security Rules（読み取り専用）

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // メニューコレクション（全ユーザー読み取り可能）
    match /menus/{menuId} {
      allow read: if true;  // 全ユーザーが読み取り可能
      allow write: if false; // 002では書き込み不可（001で管理）
    }

    // チェーンコレクション（全ユーザー読み取り可能）
    match /chains/{chainId} {
      allow read: if true;  // 全ユーザーが読み取り可能
      allow write: if false; // 002では書き込み不可（001で管理）
    }

    // お気に入りコレクション（ログインユーザーのみ読み取り可能）
    match /users/{userId}/favorites/{chainId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if false; // 002では書き込み不可（003で管理）
    }
  }
}
```

---

## Performance Considerations

### Firestore読み取り回数（Phase2想定）

#### シナリオ1: 全メニュー表示（お気に入りフィルタOFF）
- **メニュー一覧取得**: 1回のクエリ（160件程度）= 160 reads
- **チェーン名取得**: メモリキャッシュ活用（16チェーン）= 0〜16 reads（初回のみ）
- **合計**: 160〜176 reads / ページ表示

#### シナリオ2: お気に入りフィルタON（お気に入りチェーン3件の場合）
- **お気に入りチェーンID一覧**: 1回のクエリ（3件）= 3 reads
- **メニュー一覧取得**: 1回のクエリ（約30件）= 30 reads
- **チェーン名取得**: メモリキャッシュ活用（3チェーン）= 0〜3 reads（初回のみ）
- **合計**: 33〜36 reads / ページ表示

### キャッシュ戦略
- **Firestoreキャッシュ**: デフォルトで有効、ネットワーク不要時は0 reads
- **チェーン名キャッシュ**: メモリ内でキャッシュ（React state）、2回目以降はFirestore読み取り不要

### コスト試算（Phase2想定）
- **DAU**: 100人
- **1日あたりページ表示**: 100人 × 3回 = 300回
- **1日あたり読み取り**: 300回 × 160 reads = 48,000 reads
- **月間読み取り**: 48,000 reads × 30日 = 1,440,000 reads
- **Firestore無料枠**: 50,000 reads / 日 = 1,500,000 reads / 月
- **Phase2コスト**: 無料枠内

---

## Data Model（TypeScript型定義）

```typescript
// Firestore Timestamp型
import { Timestamp } from 'firebase/firestore';

// メニュー
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

// チェーン店
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

// お気に入り
export interface Favorite {
  chainId: string;
  createdAt: Timestamp;
}

// メニューステータス（クライアント側で計算）
export type MenuStatus = '予定' | '発売から◯日経過' | '発売から◯ヶ月以上経過' | '販売終了';

// メニュー表示用（UIで使用）
export interface MenuWithChain extends Menu {
  chainName: string;
  status: MenuStatus;
}
```

---

## Notes

- **読み取り専用**: 002-menu-listはデータの読み取りのみ、書き込みは001-admin-panelと003-favoritesが担当
- **001との整合性**: /menus、/chainsのスキーマは001-admin-panelと完全一致
- **003との依存**: お気に入りフィルタは003-favorites機能に依存
- **キャッシュ活用**: Firestoreキャッシュとメモリキャッシュを併用してコスト削減
