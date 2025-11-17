# ADR-003: React Admin選定理由

**Status**: 承認済み
**Date**: 2025-11-16
**Author**: 重次弘規
**Related**: [specs/001-admin-panel/research.md](../specs/001-admin-panel/research.md), [ADR-001](./001-why-firebase.md)

## Context

管理画面（limimeshi-admin）は、チェーン店と期間限定メニューを手動で登録・編集・削除する機能を提供する。

### 要件

**機能要件**:
- チェーン店のCRUD（作成・読み取り・更新、削除なし）
- メニューのCRUD（作成・読み取り・更新・削除）
- 管理者認証（Firebase Authentication + Custom Claims）
- ふりがなでの50音順ソート
- ステータス自動判定（予定、発売から◯日経過、販売終了）

**非機能要件**:
- 開発速度優先（Phase2を早期リリース）
- 保守性重視（1名運用）
- Firebase統合（Firestore、Firebase Authentication）
- Material UIベース（学習コスト低）

### 検討した選択肢

#### Option 1: React Admin 4.x（採用）
**概要**: CRUD管理画面の構築に特化したフレームワーク
- **アーキテクチャ**: フルスタック（UI + データプロバイダー）
- **UIライブラリ**: Material UI（デフォルト）
- **Firebaseサポート**: react-admin-firebase（公式推奨）
- **実績**: 25,000社利用、50万セッション/日

#### Option 2: Refine
**概要**: ヘッドレスな管理画面フレームワーク
- **アーキテクチャ**: ヘッドレス（UIは別途選択）
- **UIライブラリ**: Ant Design, Material UI, Mantine, Chakra UI
- **Firebaseサポート**: refine-firebase（週44DL）
- **実績**: GitHub 31K stars、15K本番環境

#### Option 3: 自作（React + Firestore直接）
**概要**: Reactで管理画面を完全に自作
- **アーキテクチャ**: カスタム実装
- **UIライブラリ**: 自由選択
- **Firebaseサポート**: Firebase SDK直接使用
- **実績**: なし（個人実装）

## Decision

**React Admin 4.x** を採用する

### 選定理由

#### 1. 開発速度優先（最重要）

**React Admin**:
- ✅ CRUD画面を最小限のコードで実装
- ✅ データプロバイダー（react-admin-firebase）が完成している
- ✅ フォーム検証・エラーハンドリングが標準搭載
- ✅ ボイラープレートコード最小限

**実装例**:
```typescript
// ChainList.tsx（10行程度）
export const ChainList = () => (
  <List>
    <Datagrid>
      <TextField source="name" />
      <TextField source="furigana" />
      <TextField source="officialUrl" />
      <EditButton />
    </Datagrid>
  </List>
);
```

**Refine**:
- ❌ ヘッドレスアーキテクチャで柔軟だが、ボイラープレートが多い
- ❌ UIコンポーネントを個別に実装する必要がある

**自作**:
- ❌ CRUD画面を全て自前実装（開発期間が長い）
- ❌ フォーム検証・エラーハンドリングを自前実装

#### 2. Firebase統合

**React Admin**:
- ✅ react-admin-firebase パッケージが成熟（GitHub 600+ stars）
- ✅ Firestore CRUD、認証、ファイルアップロードをサポート
- ✅ Custom Claims による権限管理をサポート

**Refine**:
- ❌ refine-firebase の成熟度が低い（週44DL）
- ❌ Custom Claimsのサポートが不明確

**自作**:
- ✅ Firebase SDK直接使用で完全制御
- ❌ データプロバイダーを自前実装（高コスト）

#### 3. 保守性（1名運用）

**React Admin**:
- ✅ 豊富なドキュメント（公式ドキュメント充実）
- ✅ 大規模コミュニティ（25,000社利用）
- ✅ 2016年から開発、2025年も活発に更新

**Refine**:
- ✅ ドキュメント充実
- ❌ React Adminより新しい（コミュニティが小さい）

**自作**:
- ❌ 全て自前で保守（1名運用では困難）

#### 4. Phase2要件への適合

**React Admin**:
- ✅ チェーン店・メニューのCRUD機能が標準
- ✅ Firebase Authentication + Custom Claims に対応
- ✅ Material UIがデフォルト（学習コスト低）

**Refine**:
- ✅ 要件に適合
- ❌ 柔軟性が高い分、Phase2では過剰

**自作**:
- ✅ 要件に完全適合
- ❌ 開発コストが高い

#### 5. 学習コスト

**React Admin**:
- ✅ Material UIベース（広く使われている）
- ✅ React Adminの公式チュートリアルが充実
- ✅ 公式デモで実装パターンを確認可能

**Refine**:
- ❌ ヘッドレスアーキテクチャの学習が必要
- ❌ UIライブラリを選択する手間

**自作**:
- ❌ React + Firebase + 管理画面設計の全てを学習

### 比較表

| 評価軸 | React Admin | Refine | 自作 |
|--------|-------------|--------|------|
| **開発速度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Firebase統合** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **保守性** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Phase2適合** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **学習コスト** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **柔軟性** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **コミュニティ** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ |

### Phase2での判断

Phase2（MVP）では以下の特性がReact Adminに有利：
- **開発速度優先**: Phase2を早期にリリース
- **保守性重視**: 1名運用、ドキュメント・コミュニティが重要
- **Firebase統合**: react-admin-firebase が成熟
- **Material UI**: 学習コスト低

## Consequences

### メリット

#### 開発速度
- CRUD画面を最小限のコードで実装（各コンポーネント10行程度）
- データプロバイダー（react-admin-firebase）が完成
- フォーム検証・エラーハンドリングが標準搭載

#### 保守性
- 豊富なドキュメント・コミュニティ
- 2016年から開発、2025年も活発に更新
- 25,000社利用の実績

#### Firebase統合
- react-admin-firebase パッケージが成熟
- Firestore CRUD、認証、ファイルアップロードをサポート
- Custom Claims による権限管理をサポート

#### Material UI
- Material UIがデフォルト（学習コスト低）
- Material UI v5（2021年）、v7（2025年）にも対応

### デメリットと対策

#### Material UI縛り
- **制約**: React AdminはMaterial UIがデフォルト、他のUIライブラリへの変更が困難
- **対策**:
  - Phase2ではMaterial UIで実装
  - Phase3以降でUI刷新が必要な場合、ra-core（ヘッドレス部分）を使って移行

#### リアルタイム更新なし
- **制約**: React Admin v3.x以降はリアルタイム更新非対応
- **対策**: Phase2では許容（管理者1名、リアルタイム更新不要）

#### SSR非対応
- **制約**: React AdminはSSR非対応（デスクトップ管理画面では不要）
- **対策**: Phase2では許容（管理画面はPC主体）

#### Shadcn UI未対応
- **制約**: 2025年にShadcn UI対応を発表したが、まだ実装されていない
- **対策**: Phase2ではMaterial UI、Phase3でShadcn UIを検討

### 実装時の注意点

#### react-admin-firebase の使用

```typescript
import {
  FirebaseDataProvider,
  FirebaseAuthProvider
} from 'react-admin-firebase';

const firebaseConfig = {
  apiKey: process.env.VITE_FIREBASE_API_KEY,
  authDomain: process.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.VITE_FIREBASE_PROJECT_ID,
  // ...
};

const dataProvider = FirebaseDataProvider(firebaseConfig);
const authProvider = FirebaseAuthProvider(firebaseConfig);
```

#### CREATE と EDIT の実装パターン

React Adminの標準設計では、CREATE と EDIT は別コンポーネントとして実装：
- **Create**: `/chains/create` → 空のフォーム → `dataProvider.create()`
- **Edit**: `/chains/:id` → 既存データ取得 → `dataProvider.update()`

公式ドキュメントより：
> "Create and Edit views almost never have exactly the same form inputs (which is a deliberate design choice)"

#### Feature-based ディレクトリ構造

React公式推奨のFeature-based構造に準拠：
```
src/
├── chains/          # チェーン店リソース
│   ├── ChainList.tsx
│   ├── ChainEdit.tsx
│   └── ChainCreate.tsx
├── menus/           # メニューリソース
│   ├── MenuList.tsx
│   ├── MenuEdit.tsx
│   └── MenuCreate.tsx
└── auth/            # 認証
    └── LoginPage.tsx
```

参考:
- [React公式 - Grouping by features or routes](https://react.dev/learn/thinking-in-react#step-5-add-inverse-data-flow)
- [React Admin公式デモ](https://github.com/marmelab/react-admin/tree/master/examples/demo/src)

### 将来の見直しタイミング

以下の状況になった場合、再評価する：
- Material UI以外のUIライブラリが必要になった場合（Shadcn UI等）
- リアルタイム更新が必須になった場合
- SSRが必要になった場合（SEO対策等）

その場合、以下の選択肢を検討：
- Refine（ヘッドレスアーキテクチャ）
- ra-core（React Adminのヘッドレス部分）+ カスタムUI
- 自作（React + Firebase直接）

## Notes

- Constitution III. Simplicity（シンプルさ優先）に従う
- ADR-001（Firebase選定）と整合
- Phase2ではReact Adminのデフォルト機能を最大限活用
- 不要なラッパー・抽象化を作らない
