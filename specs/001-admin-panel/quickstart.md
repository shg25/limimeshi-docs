# Quickstart Guide: 管理画面（Admin Panel）

**Feature**: 001-admin-panel  
**Date**: 2025-11-16  
**Status**: Phase 1 - Quickstart  
**Related**: [spec.md](./spec.md), [plan.md](./plan.md), [research.md](./research.md)  

## Overview

管理画面の開発環境セットアップ手順

## Prerequisites

### 必須ソフトウェア

- **Node.js**: 20.x LTS 以上
- **npm**: 10.x 以上（Node.jsに同梱）
- **Git**: 2.x 以上
- **Firebase CLI**: 13.x 以上
- **VSCode**: 推奨（任意のエディタ可）

### アカウント

- **Firebase**: Googleアカウント（無料プランでOK）
- **GitHub**: 開発者アカウント

---

## Setup Steps

### 1. Node.js インストール

**macOS / Linux**:
```bash
# nodenv を使用（推奨）
curl -fsSL https://github.com/nodenv/nodenv-installer/raw/master/bin/nodenv-installer | bash
nodenv install 20.11.0
nodenv global 20.11.0

# または nvm を使用
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20.11.0
nvm use 20.11.0
```

**Windows**:
- [Node.js 公式サイト](https://nodejs.org/) から LTS版をダウンロードしてインストール

**バージョン確認**:
```bash
node -v  # v20.11.0以上
npm -v   # 10.x以上
```

---

### 2. Firebase CLI インストール

```bash
npm install -g firebase-tools

# バージョン確認
firebase --version  # 13.x以上
```

**Firebase ログイン**:
```bash
firebase login
```

---

### 3. リポジトリクローン

```bash
# リポジトリをクローン（SSH推奨）
git clone git@github.com:username/limimeshi-admin.git
cd limimeshi-admin

# または HTTPS
git clone https://github.com/username/limimeshi-admin.git
cd limimeshi-admin
```

---

### 4. 依存パッケージインストール

```bash
npm install
```

**インストールされる主要パッケージ**:
- `react`: ^18.3.0
- `react-dom`: ^18.3.0
- `react-admin`: ^4.16.0
- `react-admin-firebase`: ^4.1.0
- `firebase`: ^10.13.0
- `typescript`: ^5.5.0
- `vite`: ^5.4.0

---

### 5. Firebase プロジェクト作成

**Note**: Firebase Hosting構成の詳細は [ADR-005](../../adr/005-firebase-hosting-multi-site.md) を参照

#### 5-1. Firebase Console でプロジェクト作成

1. [Firebase Console](https://console.firebase.google.com/) にアクセス
2. 「プロジェクトを追加」をクリック
3. プロジェクト名: `limimeshi-admin-dev`（開発環境）
4. Google アナリティクス: 無効（Phase2では不要）
5. 「プロジェクトを作成」をクリック

#### 5-2. Firebase プロジェクトを初期化

```bash
firebase init
```

**選択項目**:
- **Features**: Firestore, Hosting, Functions（後でセットアップ）
- **Project**: 既存のプロジェクト → `limimeshi-admin-dev` を選択
- **Firestore Rules**: `firestore.rules`（デフォルト）
- **Firestore Indexes**: `firestore.indexes.json`（デフォルト）
- **Hosting public directory**: `dist`（Viteのビルド出力先）
- **Single-page app**: Yes
- **GitHub Actions**: No（Phase2では不要）

#### 5-3. Firebase 設定を取得

1. Firebase Console → プロジェクト設定 → アプリを追加 → Web
2. アプリのニックネーム: `limimeshi-admin`
3. Firebase Hosting: 無効（後でセットアップ）
4. 「アプリを登録」をクリック
5. Firebase SDK スニペットをコピー

---

### 6. 環境変数設定

#### 6-1. `.env.local` ファイル作成

```bash
cp .env.example .env.local
```

#### 6-2. `.env.local` に Firebase 設定を追加

```bash
# .env.local
VITE_FIREBASE_API_KEY=your-api-key
VITE_FIREBASE_AUTH_DOMAIN=your-project-id.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project-id.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your-sender-id
VITE_FIREBASE_APP_ID=your-app-id
```

**Notes**:
- `.env.local` はGitにコミットしない（`.gitignore` に追加済み）
- Firebase SDK スニペットの値を `VITE_` プレフィックス付きで設定

---

### 7. Firestore セットアップ

#### 7-1. Firestore データベース作成

1. Firebase Console → Firestore Database → データベースを作成
2. セキュリティルール: **本番環境モード**（Phase2では管理者のみアクセス）
3. ロケーション: `asia-northeast1`（東京）
4. 「有効にする」をクリック

#### 7-2. Firestore Security Rules デプロイ

```bash
firebase deploy --only firestore:rules
```

**firestore.rules**:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null && request.auth.token.admin == true;
    }

    match /chains/{chainId} {
      allow read, write: if isAdmin();
      allow delete: if false;
    }

    match /menus/{menuId} {
      allow read, write, delete: if isAdmin();
    }

    match /admins/{adminId} {
      allow read: if isAdmin();
      allow write: if false;
    }
  }
}
```

#### 7-3. Firestore Indexes デプロイ

```bash
firebase deploy --only firestore:indexes
```

**firestore.indexes.json**:
```json
{
  "indexes": [
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

### 8. Firebase Authentication セットアップ

#### 8-1. Firebase Authentication 有効化

1. Firebase Console → Authentication → 始める
2. ログイン方法タブ → メール/パスワード → 有効にする
3. 「保存」をクリック

#### 8-2. 管理者ユーザー作成

**Firebase Console で手動作成**:
1. Firebase Console → Authentication → ユーザータブ → ユーザーを追加
2. メールアドレス: `admin@limimeshi.com`
3. パスワード: 任意の安全なパスワード
4. 「ユーザーを追加」をクリック

**UID をコピー**:
- 作成したユーザーのUIDをコピー（例: `abc123xyz456`）

#### 8-3. Custom Claims 設定（Firebase CLI）

```bash
# Firebase プロジェクト選択
firebase use limimeshi-admin-dev

# Custom Claims 設定（Node.js スクリプト）
node scripts/set-admin-claim.js <UID>
```

**scripts/set-admin-claim.js**:
```javascript
const admin = require('firebase-admin');

// Firebase Admin SDK初期化
admin.initializeApp();

// コマンドライン引数からUIDを取得
const uid = process.argv[2];

if (!uid) {
  console.error('Usage: node set-admin-claim.js <UID>');
  process.exit(1);
}

// Custom Claims設定
admin.auth().setCustomUserClaims(uid, { admin: true })
  .then(() => {
    console.log(`Admin claim set for user ${uid}`);
    process.exit(0);
  })
  .catch((error) => {
    console.error('Error setting admin claim:', error);
    process.exit(1);
  });
```

**実行例**:
```bash
node scripts/set-admin-claim.js abc123xyz456
# Output: Admin claim set for user abc123xyz456
```

---

### 9. 開発サーバー起動

```bash
npm run dev
```

**出力例**:
```
  VITE v5.4.0  ready in 300 ms

  ➜  Local:   http://localhost:3000/
  ➜  Network: use --host to expose
```

**ブラウザで確認**:
1. http://localhost:3000/ にアクセス
2. ログインページが表示される
3. 作成した管理者アカウント（`admin@limimeshi.com`）でログイン
4. 管理画面が表示される

---

## Development Workflow

### 開発サーバー起動

```bash
npm run dev
```

**ホットリロード**:
- ソースコードを編集すると自動でブラウザがリロードされる
- `src/` 配下のファイルを変更すると即座に反映

---

### ビルド

```bash
npm run build
```

**出力先**: `dist/`

**ビルド内容**:
- TypeScriptのコンパイル
- JSのバンドル・ミニファイ
- CSSの最適化

---

### プレビュー

ビルド後のアプリをローカルでプレビュー

```bash
npm run preview
```

**出力例**:
```
  ➜  Local:   http://localhost:4173/
```

---

### リント

```bash
npm run lint
```

**対象ファイル**: `src/**/*.ts`, `src/**/*.tsx`

---

### フォーマット

```bash
npm run format
```

**対象ファイル**: `src/**/*`（Prettierで自動フォーマット）

---

### テスト

**ユニットテスト**:
```bash
npm run test
```

**E2Eテスト**:
```bash
npm run test:e2e
```

---

## Firestore データ操作

### サンプルデータ投入

**チェーン店を追加**:
1. 開発サーバーを起動（`npm run dev`）
2. http://localhost:3000/ にアクセス
3. 「Chains」メニューをクリック
4. 「CREATE」ボタンをクリック
5. フォームに入力:
   - Name: `マクドナルド`
   - Furigana: `まくどなるど`
   - Official URL: `https://www.mcdonalds.co.jp/`
6. 「SAVE」をクリック

**メニューを追加**:
1. 「Menus」メニューをクリック
2. 「CREATE」ボタンをクリック
3. フォームに入力:
   - Chain: `マクドナルド` を選択
   - Name: `てりたま`
   - Description: `たまごとテリヤキソースの期間限定バーガー`
   - Price: `390`
   - Sale Start Time: `2025-11-01 00:00`
   - Sale End Time: `2025-11-30 23:59`
4. 「SAVE」をクリック

---

## Troubleshooting

### ログイン失敗（Custom Claims未設定）

**症状**:
```
Error: Not authorized as admin
```

**解決方法**:
```bash
# Custom Claims設定スクリプトを再実行
node scripts/set-admin-claim.js <UID>

# トークンをリフレッシュ（ログアウト→ログイン）
```

---

### Firestore権限エラー

**症状**:
```
FirebaseError: Missing or insufficient permissions
```

**解決方法**:
```bash
# Firestore Security Rulesを再デプロイ
firebase deploy --only firestore:rules

# ログアウト→ログイン
```

---

### ビルドエラー（TypeScript型エラー）

**症状**:
```
error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
```

**解決方法**:
- TypeScriptの型定義を確認
- `src/` 配下のファイルを修正
- `npm run build` で再ビルド

---

### 開発サーバーが起動しない

**症状**:
```
Error: EADDRINUSE: address already in use :::3000
```

**解決方法**:
```bash
# ポート3000を使用しているプロセスを終了
lsof -ti:3000 | xargs kill -9

# または別のポートを使用
npm run dev -- --port 3001
```

---

## Next Steps

1. **Phase 1 完了確認**:
   - `data-model.md` を確認
   - `contracts/` を確認
   - `quickstart.md` を確認（このファイル）

2. **Phase 2: タスク生成**:
   - `/speckit-tasks` コマンドで `tasks.md` を生成

3. **Phase 3: 実装開始**:
   - `/speckit-implement` コマンドで実装開始

---

## Notes

- **Phase2 MVP範囲**: チェーン店管理、メニュー管理、管理者認証
- **Out of Scope**: タグ管理、入力補助機能、チェーン店削除機能
- **開発環境**: `limimeshi-admin-dev`（Firebase プロジェクト）
- **本番環境**: Phase3以降でセットアップ
- **CI/CD**: Phase3以降で GitHub Actions をセットアップ
