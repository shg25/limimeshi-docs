# Firestoreセキュリティルール設計

**作成日**: 2025-11-19  
**ベストプラクティス**: [firestore-best-practices.md](./firestore-best-practices.md)  
**関連仕様**: [001-admin-panel](../specs/001-admin-panel/), [002-chain-list](../specs/002-chain-list/), [003-favorites](../specs/003-favorites/)  
  
---

## 概要

本プロジェクトで使用するFirestoreセキュリティルール。最小権限の原則に基づき、各機能に必要な権限のみを付与する。

### 権限マトリクス

| コレクション | 一般ユーザー | ログインユーザー | 管理者 |
|-------------|-------------|----------------|--------|
| /chains | read | read | read, create, update |
| /campaigns | read | read | read, create, update, delete |
| /admins | - | - | read |
| /users/{userId}/favorites | - | read, write（自分のみ） | - |

---

## 完全なセキュリティルール

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ============================================
    // ヘルパー関数
    // ============================================

    // ログイン済みか確認
    function isAuthenticated() {
      return request.auth != null;
    }

    // 管理者か確認（Custom Claims）
    function isAdmin() {
      return isAuthenticated() && request.auth.token.admin == true;
    }

    // 本人か確認
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }

    // ============================================
    // /chains - チェーン店マスタ
    // ============================================
    match /chains/{chainId} {
      // 全ユーザーが読み取り可能
      allow read: if true;

      // 管理者のみ作成・更新可能（削除不可）
      allow create, update: if isAdmin();

      // 削除は禁止（Phase2では削除機能なし）
      allow delete: if false;
    }

    // ============================================
    // /campaigns - キャンペーン
    // ============================================
    match /campaigns/{campaignId} {
      // 全ユーザーが読み取り可能
      allow read: if true;

      // 管理者のみ作成・更新・削除可能
      allow create, update, delete: if isAdmin();
    }

    // ============================================
    // /admins - 管理者情報
    // ============================================
    match /admins/{adminId} {
      // 管理者のみ読み取り可能
      allow read: if isAdmin();

      // 書き込みはFirebase Admin SDKのみ（クライアントからは不可）
      allow write: if false;
    }

    // ============================================
    // /users/{userId}/favorites - お気に入りチェーン
    // ============================================
    match /users/{userId}/favorites/{chainId} {
      // 本人のみ読み取り可能
      allow read: if isOwner(userId);

      // 本人のみ作成・削除可能（更新は不要）
      allow create, delete: if isOwner(userId);

      // 更新は不要（登録or解除のみ）
      allow update: if false;
    }

    // ============================================
    // /users/{userId} - ユーザー情報（Phase3以降）
    // ============================================
    match /users/{userId} {
      // 本人のみ読み書き可能
      allow read, write: if isOwner(userId);
    }

    // ============================================
    // /reviews - レビュー（Phase3以降）
    // ============================================
    match /reviews/{reviewId} {
      // 全ユーザーが読み取り可能
      allow read: if true;

      // ログインユーザーのみ作成可能
      allow create: if isAuthenticated()
                    && request.resource.data.userId == request.auth.uid;

      // 本人のみ更新・削除可能
      allow update, delete: if isAuthenticated()
                            && resource.data.userId == request.auth.uid;
    }
  }
}
```

---

## ルール詳細

### /chains

| 操作 | 条件 | 理由 |
|------|------|------|
| read | 全ユーザー | 002-chain-listでチェーン名表示に必要 |
| create | 管理者のみ | 001-admin-panelで登録 |
| update | 管理者のみ | 001-admin-panelで編集、003-favoritesでfavoriteCount更新 |
| delete | 禁止 | Phase2では削除機能なし、誤削除防止 |

**注意**: `favoriteCount`の更新は、003-favoritesのTransactionで実行される。セキュリティルール上は`update`を許可するが、クライアント側でTransactionを使用して整合性を保証する。

### /campaigns

| 操作 | 条件 | 理由 |
|------|------|------|
| read | 全ユーザー | 002-chain-listでキャンペーン一覧表示 |
| create | 管理者のみ | 001-admin-panelで登録 |
| update | 管理者のみ | 001-admin-panelで編集 |
| delete | 管理者のみ | 001-admin-panelで削除 |

### /admins

| 操作 | 条件 | 理由 |
|------|------|------|
| read | 管理者のみ | 管理画面ログイン時に自身の情報確認 |
| write | 禁止 | Firebase Admin SDK（サーバーサイド）からのみ作成 |

### /users/{userId}/favorites

| 操作 | 条件 | 理由 |
|------|------|------|
| read | 本人のみ | 他ユーザーのお気に入りは参照不要 |
| create | 本人のみ | お気に入り登録 |
| delete | 本人のみ | お気に入り解除 |
| update | 禁止 | 登録/解除のみ、更新操作は不要 |

---

## バリデーションルール（拡張版）

データの整合性を保証するため、書き込み時のバリデーションを追加する。

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ヘルパー関数（同上）
    function isAuthenticated() {
      return request.auth != null;
    }

    function isAdmin() {
      return isAuthenticated() && request.auth.token.admin == true;
    }

    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }

    // バリデーション関数
    function isValidChain() {
      let data = request.resource.data;
      return data.name is string
          && data.name.size() >= 1
          && data.name.size() <= 100
          && data.furigana is string
          && data.furigana.size() >= 1
          && data.furigana.size() <= 100
          && data.favoriteCount is int
          && data.favoriteCount >= 0;
    }

    function isValidCampaign() {
      let data = request.resource.data;
      return data.chainId is string
          && data.name is string
          && data.name.size() >= 1
          && data.name.size() <= 100
          && data.saleStartTime is timestamp;
    }

    function isValidFavorite() {
      let data = request.resource.data;
      return data.chainId is string
          && data.createdAt is timestamp;
    }

    // /chains
    match /chains/{chainId} {
      allow read: if true;
      allow create: if isAdmin() && isValidChain();
      allow update: if isAdmin();
      allow delete: if false;
    }

    // /campaigns
    match /campaigns/{campaignId} {
      allow read: if true;
      allow create: if isAdmin() && isValidCampaign();
      allow update, delete: if isAdmin();
    }

    // /admins
    match /admins/{adminId} {
      allow read: if isAdmin();
      allow write: if false;
    }

    // /users/{userId}/favorites
    match /users/{userId}/favorites/{chainId} {
      allow read: if isOwner(userId);
      allow create: if isOwner(userId) && isValidFavorite();
      allow delete: if isOwner(userId);
      allow update: if false;
    }
  }
}
```

---

## テスト方法

### Firebase Emulatorでのテスト

```bash
# エミュレーター起動
firebase emulators:start --only firestore

# ルールテスト実行
firebase emulators:exec --only firestore "npm run test:rules"
```

### テストケース

| テストケース | 期待結果 |
|-------------|---------|
| 未認証ユーザーが/chainsを読み取り | 許可 |
| 未認証ユーザーが/campaignsを読み取り | 許可 |
| 未認証ユーザーが/chainsに書き込み | 拒否 |
| 管理者が/chainsに書き込み | 許可 |
| 管理者が/campaignsに書き込み | 許可 |
| ログインユーザーが自分のfavoritesを読み取り | 許可 |
| ログインユーザーが他人のfavoritesを読み取り | 拒否 |
| ログインユーザーがfavoritesを作成 | 許可 |
| ログインユーザーがfavoritesを更新 | 拒否 |

---

## デプロイ

### Firebase CLIでのデプロイ

```bash
# ルールファイルをデプロイ
firebase deploy --only firestore:rules
```

### ルールファイルの場所

```
limimeshi-infra/
├── firestore.rules    # セキュリティルール
├── firestore.indexes.json  # インデックス設定
└── firebase.json      # Firebase設定
```

---

## 参考

- [Firestore設計ベストプラクティス](./firestore-best-practices.md)
- [セキュリティルールの構造（公式）](https://firebase.google.com/docs/firestore/security/rules-structure)
- [セキュリティルールのベストプラクティス（公式）](https://firebase.google.com/docs/firestore/security/get-started)
