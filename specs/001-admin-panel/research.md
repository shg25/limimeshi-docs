# Research: 管理画面フレームワーク選定

**Feature**: 001-admin-panel  
**Date**: 2025-11-16  
**Status**: Phase 0 - Technology Decision  

## 1. 管理画面フレームワークの選定

### Decision

**React Admin 4.x** を採用する

### Rationale

以下の3つの選択肢を評価した結果、React Admin が本プロジェクトの要件に最も適していると判断：

#### 評価軸と各選択肢の比較

| 評価軸 | React Admin 4.x | Refine | 自作（React + Firestore） |
|--------|----------------|--------|-------------------------|
| **開発速度** | ⭐⭐⭐⭐⭐ 最速 | ⭐⭐⭐⭐ 速い | ⭐⭐ 遅い |
| **保守性** | ⭐⭐⭐⭐ 高い | ⭐⭐⭐⭐ 高い | ⭐⭐ 低い（自前で保守） |
| **Firebase統合** | ⭐⭐⭐⭐ react-admin-firebase | ⭐⭐⭐ refine-firebase | ⭐⭐⭐⭐⭐ 完全制御 |
| **学習コスト** | ⭐⭐⭐⭐ 低い | ⭐⭐⭐ 中程度 | ⭐⭐ 高い |
| **Phase2要件適合** | ⭐⭐⭐⭐⭐ 完全適合 | ⭐⭐⭐⭐ 適合 | ⭐⭐⭐ 適合（自作必要） |
| **柔軟性** | ⭐⭐⭐ 中程度 | ⭐⭐⭐⭐⭐ 非常に高い | ⭐⭐⭐⭐⭐ 完全制御 |
| **コミュニティ** | ⭐⭐⭐⭐⭐ 25,000社利用 | ⭐⭐⭐⭐ 31K stars, 15K本番 | ⭐ なし |

### 選定理由の詳細

#### React Admin を選んだ主な理由

1. **Phase2 MVPの要件に完全適合**
   - チェーン店とメニューのCRUD機能が標準で提供
   - Firebase Authentication + Custom Claims に対応
   - Firestore データプロバイダーが利用可能

2. **個人開発での開発速度優先**
   - ボイラープレートコードが最小限
   - CRUD画面の自動生成
   - Phase2を早期にリリースすることが重要

3. **成熟したエコシステム**
   - 2016年から開発、50万セッション/日の実績
   - 豊富なドキュメントとコミュニティサポート
   - 2025年も活発に更新（Material UI v7対応等）

4. **Firebase統合の実績**
   - `react-admin-firebase` パッケージが安定稼働
   - Firestore の CRUD、認証、ファイルアップロードに対応
   - Custom Claims による権限管理をサポート

5. **Material UIとの親和性**
   - Material UI がデフォルト（学習コスト低）
   - 2025年にShadcn UI対応も発表（将来の選択肢）

### Alternatives Considered

#### Refine（検討したが不採用）

**長所**:
- ヘッドレスアーキテクチャで柔軟性が高い
- 複数UIフレームワーク対応（Ant Design, Material UI, Mantine, Chakra UI）
- SSRサポート（Next.js, Remix）
- 全機能がOSS（Enterprise版なし）
- AI支援ツール（RefineAI）

**短所（本プロジェクトでの不採用理由）**:
- React Admin より多くのボイラープレートが必要
- 柔軟性は高いが、Phase2 MVPでは過剰
- Firebase統合パッケージの成熟度が React Admin より低い（週44DL）
- 個人開発では「速く作る」方が重要、柔軟性は二の次

**判断**: Phase3以降でUI刷新が必要になった場合に再検討

#### 自作（React + Firestore直接）（検討したが不採用）

**長所**:
- 完全なカスタマイズ性
- Firestoreとの統合を完全制御
- 余計な抽象化なし

**短所（本プロジェクトでの不採用理由）**:
- 開発コストが非常に高い（CRUD画面を全て実装）
- 認証フロー、フォーム検証、エラーハンドリングを自前実装
- 保守コストが高い（1名運用では非現実的）
- Phase2 MVPのリリース時期が大幅に遅延

**判断**: MVP段階で自作は不合理、実装後の保守も困難

### Implementation Notes

#### react-admin-firebase パッケージの使用

```typescript
import {
  FirebaseDataProvider,
  FirebaseAuthProvider
} from 'react-admin-firebase';

// Firebase設定
const firebaseConfig = {
  apiKey: process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain: process.env.REACT_APP_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.REACT_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.REACT_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.REACT_APP_FIREBASE_APP_ID
};

const dataProvider = FirebaseDataProvider(firebaseConfig);
const authProvider = FirebaseAuthProvider(firebaseConfig);
```

#### Phase2での制約事項

- **リアルタイム更新なし**: React Admin v3.x以降は非対応（Phase2では許容）
- **Material UI縛り**: Phase2ではMaterial UIのみ使用（Shadcn UIはPhase3で検討）
- **SSR非対応**: デスクトップ管理画面では不要

#### Phase3以降の拡張性

- ra-core（ヘッドレス部分）により、将来的にUI刷新可能
- 必要に応じて Refine や自作への移行も選択肢として残る

---

## 2. Firestore Data Provider for React Admin

### Decision

**react-admin-firebase** パッケージを使用する

### Rationale

- 公式にReact Adminのドキュメントに掲載されている
- GitHub stars 600+、成熟したパッケージ
- Firestore の全CRUD操作をサポート
- Firebase Authentication との統合が完成している

### Alternatives Considered

- **カスタムデータプロバイダー**: 実装コストが高く、Phase2では不要
- **他のFirestoreプロバイダー**: react-admin-firebase が最も成熟

### Implementation Notes

```typescript
// Firestoreコレクション構造
/chains/{chainId}
/campaigns/{campaignId}

// react-admin-firebase は自動的にコレクションをマッピング
<Resource name="chains" list={ChainList} edit={ChainEdit} create={ChainCreate} />
<Resource name="campaigns" list={CampaignList} edit={CampaignEdit} create={CampaignCreate} />
```

**注意点**:
- サブコレクションの管理は app config で設定可能
- Firestore の random ID を "id" フィールドでオーバーライド可能
- FileInput による Firebase Storage へのアップロードをサポート

#### CREATE と EDIT の実装パターン

React Adminの標準設計では、CREATE と EDIT は別コンポーネントとして実装する：

- **Create**: `/chains/create` → 空のフォーム → `dataProvider.create()`
- **Edit**: `/chains/:id` → 既存データ取得 → `dataProvider.update()`

公式ドキュメントより：
> "Create and Edit views almost never have exactly the same form inputs (which is a deliberate design choice)"

**実装例**:

```typescript
// ChainCreate.tsx
export const ChainCreate = () => (
  <Create>
    <SimpleForm>
      <TextInput source="name" validate={required()} />
      <TextInput source="furigana" validate={required()} />
      <TextInput source="officialUrl" />
      <TextInput source="logoUrl" />
    </SimpleForm>
  </Create>
);

// ChainEdit.tsx
export const ChainEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="name" validate={required()} />
      <TextInput source="furigana" validate={required()} />
      <TextInput source="officialUrl" />
      <TextInput source="logoUrl" />
      <NumberField source="favoriteCount" disabled />
    </SimpleForm>
  </Edit>
);
```

**本プロジェクトの方針**:
- React Adminの標準パターンに従い、ChainCreate/ChainEdit、CampaignCreate/CampaignEdit を個別に実装
- コード量は各コンポーネント10行程度で済む
- 必要に応じてカスタムFormコンポーネントで共通化可能

---

## 3. Firebase Authentication with Custom Claims

### Decision

Firebase Authentication + Custom Claims で管理者権限を管理

### Rationale

- Firebaseの標準機能で実装可能
- Custom Claims（`{ admin: true }`）で管理者を識別
- Cloud Functions で Custom Claims を設定
- Firestore Security Rules で権限チェック

### Implementation Notes

#### Cloud Functions で Custom Claims 設定

```typescript
// functions/src/setAdminClaim.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const setAdminClaim = functions.https.onCall(async (data, context) => {
  // 既存の管理者のみが実行可能
  if (!context.auth?.token.admin) {
    throw new functions.https.HttpsError(
      'permission-denied',
      'Only admins can set admin claims.'
    );
  }

  const { uid } = data;
  await admin.auth().setCustomUserClaims(uid, { admin: true });
  return { message: `Admin claim set for user ${uid}` };
});
```

#### Firestore Security Rules で権限チェック

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null && request.auth.token.admin == true;
    }

    match /chains/{chainId} {
      allow read, write: if isAdmin();
    }

    match /campaigns/{campaignId} {
      allow read, write: if isAdmin();
    }
  }
}
```

#### React Admin での認証チェック

```typescript
const authProvider = FirebaseAuthProvider(firebaseConfig, {
  // admin Custom Claim を持つユーザーのみログイン可能
  checkAuth: async (params) => {
    const user = auth.currentUser;
    if (!user) throw new Error('Not authenticated');

    const token = await user.getIdTokenResult();
    if (!token.claims.admin) {
      throw new Error('Not authorized as admin');
    }
  }
});
```

**Phase2での初期管理者作成**:
1. Firebase Console で最初の管理者ユーザーを手動作成
2. Firebase CLI で Custom Claim を設定:
   ```bash
   firebase auth:import users.json --hash-algo=scrypt
   ```
3. または Cloud Functions を手動実行

---

## 4. Firestore Security Rules for Admin Panel

### Decision

Custom Claims を使った権限チェックを Firestore Security Rules で実装

### Rationale

- サーバーサイドで権限を強制（クライアント側のチェックは回避可能）
- Firebase の標準機能で実装可能
- 管理者以外のアクセスを完全にブロック

### Implementation Notes

上記「3. Firebase Authentication with Custom Claims」のセクション参照

**追加の考慮事項**:
- Phase2は1名運用のため、複雑な権限管理は不要
- Phase3以降で複数管理者に拡張する場合、role-based access control（RBAC）を検討
- 監査ログ（Audit log）はPhase3以降で実装

---

## 5. Vite + React + TypeScript Setup

### Decision

Vite を使用した React + TypeScript 環境

### Rationale

- Vite は最速のビルドツール（HMR が高速）
- React Admin 4.x は Vite をサポート
- TypeScript で型安全性を確保
- ESLint + Prettier で品質担保

### Implementation Notes

#### package.json

```json
{
  "name": "limimeshi-admin",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext ts,tsx",
    "format": "prettier --write src"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-admin": "^4.16.0",
    "react-admin-firebase": "^4.1.0",
    "firebase": "^10.13.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "@vitejs/plugin-react": "^4.3.0",
    "eslint": "^8.57.0",
    "eslint-config-prettier": "^9.1.0",
    "prettier": "^3.2.0",
    "typescript": "^5.5.0",
    "vite": "^5.4.0"
  }
}
```

#### vite.config.ts

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  }
});
```

#### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "strict": true,
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**推奨プラグイン・ツール**:
- ESLint: コード品質チェック
- Prettier: コードフォーマット
- Vitest: ユニットテスト（React Testing Library と併用）
- Playwright: E2Eテスト

---

## Phase 0 まとめ

### 技術スタック確定

| カテゴリ | 選定技術 | 理由 |
|---------|---------|------|
| **管理画面フレームワーク** | React Admin 4.x | 開発速度、成熟度、Firebase統合 |
| **Firestoreプロバイダー** | react-admin-firebase | 公式推奨、成熟したパッケージ |
| **認証** | Firebase Auth + Custom Claims | 標準機能、セキュア |
| **ビルドツール** | Vite | 最速、React Admin対応 |
| **言語** | TypeScript | 型安全性、保守性 |
| **テスト** | Vitest + Playwright | モダンなテストツール |
| **Hosting構成** | Firebase Hosting Multi-Site | 詳細は [ADR-005](../../adr/005-firebase-hosting-multi-site.md) 参照 |

### Next Steps（Phase 1）

1. data-model.md 作成：Firestoreのスキーマ詳細化
2. contracts/ 作成：Firestoreスキーマと React Admin データプロバイダーインターフェース
3. quickstart.md 作成：開発環境セットアップ手順
4. CLAUDE.md 更新：技術スタックを追加（update-agent-context.sh実行）

### ADR作成

この選定内容を `adr/003-admin-framework.md` として記録する（Phase1で実施）。
