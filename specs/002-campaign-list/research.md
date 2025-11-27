# Research: メニュー一覧（Menu List）

**Feature**: 002-menu-list  
**Date**: 2025-11-18  
**Status**: Phase 0 - Research  

## Overview

メニュー一覧機能の実装に必要な技術選定と設計パターンの調査結果

## 1. React + TypeScript（フロントエンド）

### Decision
React 18 + TypeScript + Vite を使用

### Rationale
- **React 18**: Constitution で React を採用済み
- **TypeScript**: Constitution VII で型安全性の確保を要求
- **Vite**: 高速な開発体験、HMR（Hot Module Replacement）、Phase1で選定予定のビルドツール

### Alternatives Considered
- **Next.js**: SSR（Server Side Rendering）が不要なため過剰（Phase2はSPA、SEO対策はPhase3以降）
- **Create React App**: Vite より遅い、メンテナンス終了の懸念

### Reference
- [React公式](https://react.dev/)
- [Vite公式](https://vitejs.dev/)

---

## 2. Firestore SDK（データ読み取り）

### Decision
Firebase JS SDK v9+ のモジュラー形式を使用

### Rationale
- **Tree-shaking対応**: バンドルサイズ削減（Constitution VI: Mobile & Performance First）
- **TypeScript完全対応**: 型安全性の確保
- **オフライン対応**: キャッシュ機能が標準搭載
- **リアルタイム更新**: `onSnapshot` でリアルタイムリスニング可能（Phase3以降で活用）

### Key APIs
```typescript
import { collection, query, where, orderBy, limit, getDocs, Timestamp } from 'firebase/firestore';

// メニュー一覧取得（1年以内、販売開始日時降順）
const menusRef = collection(db, 'menus');
const oneYearAgo = Timestamp.fromDate(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000));

const q = query(
  menusRef,
  where('saleStartTime', '>=', oneYearAgo),
  orderBy('saleStartTime', 'desc')
);

const snapshot = await getDocs(q);
```

### Firestore Indexes
以下の複合インデックスが必要：
- `saleStartTime` (descending)

お気に入りフィルタ用（chainIdの配列でフィルタ）：
- `chainId` (ascending) + `saleStartTime` (descending)

### Alternatives Considered
- **Realtime Database**: NoSQLだがクエリ機能が弱い、Firestoreが推奨
- **Cloud Functions経由**: Phase2ではオーバーエンジニアリング

### Reference
- [Firestore公式](https://firebase.google.com/docs/firestore)
- [Firestore Queries](https://firebase.google.com/docs/firestore/query-data/queries)

---

## 3. Firebase Authentication（ログインユーザー判定）

### Decision
Firebase Authentication の `onAuthStateChanged` を使用

### Rationale
- **リアルタイム認証状態監視**: ログイン/ログアウトを即座に検知
- **匿名認証対応**: Constitution で匿名認証 → Googleログインの移行を想定
- **型安全**: TypeScript完全対応

### Key APIs
```typescript
import { getAuth, onAuthStateChanged, User } from 'firebase/auth';

const auth = getAuth();

onAuthStateChanged(auth, (user: User | null) => {
  if (user) {
    // ログイン済み
    console.log('Logged in:', user.uid);
  } else {
    // 未ログイン
    console.log('Not logged in');
  }
});
```

### Alternatives Considered
- **Custom Auth**: Phase2ではFirebase Authで十分

### Reference
- [Firebase Authentication](https://firebase.google.com/docs/auth)

---

## 4. X（Twitter）Post埋め込み

### Decision
X（Twitter）公式の `oEmbed API` + `react-twitter-embed` ライブラリを使用

### Rationale
- **公式API**: 利用規約遵守、埋め込み表示が公式に許可されている
- **react-twitter-embed**: React向けの軽量ライブラリ、TypeScript対応
- **レスポンシブ対応**: モバイル表示に最適化
- **画像自動表示**: Postに画像が添付されていれば自動的に埋め込まれる

### Installation
```bash
npm install react-twitter-embed
```

### Usage Example
```tsx
import { TwitterTweetEmbed } from 'react-twitter-embed';

// X Post URLからPost IDを抽出
const extractTweetId = (url: string): string | null => {
  const match = url.match(/status\/(\d+)/);
  return match ? match[1] : null;
};

// 埋め込み表示
const tweetId = extractTweetId(menu.xPostUrl);
if (tweetId) {
  <TwitterTweetEmbed tweetId={tweetId} />
}
```

### Edge Cases
- **Post削除**: Post作成者が削除した場合、埋め込み表示できなくなる（外部依存の制約として受け入れる）
- **画像なし**: 画像がない場合でもPostテキストのみ表示される
- **読み込み遅延**: X APIの応答が遅い場合があるが、ページ本体の読み込みとは分離される

### Alternatives Considered
- **手動iframe**: oEmbed APIの方が簡単で推奨
- **画像のみ保存**: 著作権リスク、利用規約違反の可能性

### Reference
- [Twitter oEmbed API](https://developer.twitter.com/en/docs/twitter-for-websites/embedded-tweets/overview)
- [react-twitter-embed](https://www.npmjs.com/package/react-twitter-embed)

---

## 5. フィルタ選択の永続化

### Decision
`localStorage` を使用してフィルタ選択（全て表示/お気に入りのみ表示）を保存

### Rationale
- **ブラウザ標準API**: 追加ライブラリ不要、シンプル
- **永続化**: ブラウザを閉じても設定が保持される
- **ログインユーザー専用**: Phase2ではログインユーザーのみフィルタ機能を提供

### Key Pattern
```typescript
// フィルタ設定の保存
const saveFilterPreference = (showFavoritesOnly: boolean) => {
  localStorage.setItem('menuFilter', JSON.stringify({ showFavoritesOnly }));
};

// フィルタ設定の読み込み
const loadFilterPreference = (): boolean => {
  const saved = localStorage.getItem('menuFilter');
  if (saved) {
    const { showFavoritesOnly } = JSON.parse(saved);
    return showFavoritesOnly;
  }
  return false; // デフォルトは全て表示
};
```

### Alternatives Considered
- **Firestore保存**: Phase2では不要（ユーザーごとに設定を保存するとFirestore読み取り回数が増える）
- **sessionStorage**: ブラウザを閉じると消えるため不適切
- **Cookie**: localStorageの方がシンプル

### Reference
- [MDN localStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage)

---

## 6. Firestoreクエリ設計

### Decision
クライアント側で複合クエリを実行し、必要に応じてキャッシュを活用

### Query Patterns

#### パターン1: 全メニュー表示（1年以内、販売開始日時降順）
```typescript
const oneYearAgo = Timestamp.fromDate(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000));

const q = query(
  collection(db, 'menus'),
  where('saleStartTime', '>=', oneYearAgo),
  orderBy('saleStartTime', 'desc')
);
```

**必要なインデックス**:
- `saleStartTime` (descending)

#### パターン2: お気に入りフィルタ（chainIdの配列でフィルタ）
```typescript
// お気に入りチェーンIDを取得（003-favoritesの機能）
const favoriteChainIds = await getFavoriteChainIds(user.uid);

// chainIdが配列に含まれるメニューを取得
const q = query(
  collection(db, 'menus'),
  where('chainId', 'in', favoriteChainIds),
  where('saleStartTime', '>=', oneYearAgo),
  orderBy('saleStartTime', 'desc')
);
```

**制約**:
- `where('chainId', 'in', ...)` は最大10個まで
- お気に入りチェーンが10個を超える場合は、クライアント側でフィルタリング

**必要なインデックス**:
- `chainId` (ascending) + `saleStartTime` (descending)

### Performance Considerations
- **キャッシュ活用**: Firestoreのキャッシュ機能を有効化（デフォルトで有効）
- **ページネーション**: Phase2では不要（メニュー数160件程度）、500件超えたら検討

### Alternatives Considered
- **Cloud Functions経由**: Phase2ではオーバーエンジニアリング
- **Realtime Listener**: Phase2では不要（リアルタイム更新はPhase3以降）

---

## 7. ステータス自動判定ロジック

### Decision
クライアント側で販売開始日時と販売終了日時を元にステータスを自動判定

### Logic
```typescript
type MenuStatus = '予定' | '発売から◯日経過' | '発売から◯ヶ月以上経過' | '販売終了';

const getMenuStatus = (saleStartTime: Timestamp, saleEndTime?: Timestamp): MenuStatus => {
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
  const daysPassed = Math.floor((now.getTime() - startDate.getTime()) / (1000 * 60 * 60 * 24));

  // 30日以内
  if (daysPassed < 30) {
    return `発売から${daysPassed}日経過`;
  }

  // 30日以上
  const monthsPassed = Math.floor(daysPassed / 30);
  return `発売から${monthsPassed}ヶ月以上経過`;
};
```

### Rationale
- **シンプル**: Firestoreに保存せず、表示時に計算
- **リアルタイム**: 常に最新の状態を表示
- **コスト削減**: Firestore書き込み不要

### Alternatives Considered
- **Firestore保存**: 日次バッチでステータスを更新する方法（オーバーエンジニアリング、コスト増）

---

## 8. UI/UXパターン

### Decision
Material-UI（MUI）をUIライブラリとして採用

### Rationale
- **React公式推奨**: Reactエコシステムで最も人気
- **モバイルファースト**: レスポンシブデザインが標準
- **TypeScript完全対応**: 型安全性の確保
- **アクセシビリティ**: WCAG 2.1 AA準拠
- **カスタマイズ性**: テーマシステムで柔軟なカスタマイズ

### Key Components
- **Card**: メニューカード表示
- **Grid/Stack**: レイアウト
- **Switch/Checkbox**: お気に入りフィルタのトグル
- **Typography**: テキスト表示
- **Skeleton**: ローディング状態

### Installation
```bash
npm install @mui/material @emotion/react @emotion/styled
```

### Alternatives Considered
- **Chakra UI**: MUIの方が成熟している
- **Ant Design**: MUIの方がモバイルフレンドリー
- **Tailwind CSS**: MUIの方がコンポーネントベースで開発しやすい

### Reference
- [Material-UI公式](https://mui.com/)

---

## 9. 状態管理

### Decision
Phase2では `useState` + `useEffect` のみ使用、グローバル状態管理ライブラリは不使用

### Rationale
- **シンプル**: Constitution III: Simplicity（YAGNI原則）
- **Phase2の規模**: メニュー一覧とお気に入りフィルタのみで、状態は局所的
- **オーバーエンジニアリング回避**: Redux/Zustand等は不要

### Pattern
```typescript
const [menus, setMenus] = useState<Menu[]>([]);
const [loading, setLoading] = useState<boolean>(true);
const [showFavoritesOnly, setShowFavoritesOnly] = useState<boolean>(false);

useEffect(() => {
  // Firestoreからメニューを取得
  fetchMenus();
}, [showFavoritesOnly]);
```

### Phase3以降の検討
- **Zustand**: 軽量、TypeScript対応、学習コスト低い
- **TanStack Query（React Query）**: Firestoreキャッシュと併用

### Alternatives Considered
- **Redux**: Phase2では過剰
- **Context API**: Phase2では不要

---

## 10. テスト戦略

### Decision
- **単体テスト**: Vitest + React Testing Library
- **E2Eテスト**: Playwright

### Rationale
- **Vitest**: Vite公式のテストフレームワーク、高速
- **React Testing Library**: React公式推奨、ユーザー目線のテスト
- **Playwright**: クロスブラウザ対応、Firestoreエミュレータと連携

### Test Coverage
- **単体テスト**:
  - ステータス判定ロジック（`getMenuStatus`）
  - フィルタロジック（お気に入りのみ表示）
  - 1年フィルタロジック
- **E2Eテスト**:
  - メニュー一覧表示
  - お気に入りフィルタのON/OFF
  - フィルタ選択の永続化

### Reference
- [Vitest公式](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright公式](https://playwright.dev/)

---

## Summary

| 項目 | 選定技術 | 理由 |
|------|---------|------|
| フロントエンド | React 18 + TypeScript + Vite | Constitution準拠、高速開発 |
| データ読み取り | Firebase JS SDK v9+ | Tree-shaking、TypeScript対応 |
| 認証 | Firebase Authentication | リアルタイム認証状態監視 |
| X Post埋め込み | react-twitter-embed | 公式API、レスポンシブ対応 |
| フィルタ永続化 | localStorage | ブラウザ標準API、シンプル |
| UIライブラリ | Material-UI（MUI） | モバイルファースト、アクセシビリティ |
| 状態管理 | useState + useEffect | YAGNI原則、Phase2では十分 |
| テスト | Vitest + Playwright | Vite公式、クロスブラウザ対応 |

全ての技術選定は Constitution に準拠しています。
