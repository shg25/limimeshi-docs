# Firebase 本番環境セットアップガイド

本番環境（limimeshi-prod）のセットアップ手順

## 前提条件

- Firebase CLI インストール済み（`npm install -g firebase-tools`）
- Google Cloud SDK インストール済み（Terraform使用時）
- 開発環境（limimeshi-dev）が稼働中

## セットアップ概要

| 手順 | 手動 | 自動化（IaC） |
|------|------|---------------|
| 1. プロジェクト作成 | Firebase Console | Terraform |
| 2. Blazeプラン切り替え | Firebase Console | ❌ 不可 |
| 3. Authentication設定 | Firebase Console | Terraform（部分的） |
| 4. Firestore有効化 | Firebase Console | Terraform |
| 5. Firestore Rules/Indexes | Firebase CLI | Firebase CLI |
| 6. 管理者Custom Claims | スクリプト | スクリプト |
| 7. Hosting設定 | Firebase CLI | Firebase CLI |
| 8. Android設定 | Firebase Console | ❌ 不可 |

---

## 方法A: 手動セットアップ（推奨）

### 1. Firebase プロジェクト作成

1. [Firebase Console](https://console.firebase.google.com/) にアクセス
2. 「プロジェクトを追加」をクリック
3. プロジェクト名: `limimeshi-prod`
4. Google Analytics: 有効化（推奨）
5. 「プロジェクトを作成」をクリック

### 2. Blazeプランへ切り替え

1. Firebase Console → プロジェクト設定 → 使用量と請求額
2. 「プランを変更」→「Blaze（従量制）」を選択
3. 支払い情報を入力

**注意**: Multi-site Hosting に必要

### 3. Authentication 設定

1. Firebase Console → Authentication → Sign-in method
2. 「メール/パスワード」を有効化
3. 管理者ユーザーを作成:
   - Users タブ → 「ユーザーを追加」
   - メールアドレスとパスワードを入力

### 4. Firestore 有効化

1. Firebase Console → Firestore Database
2. 「データベースを作成」をクリック
3. 本番環境モードを選択
4. ロケーション: `asia-northeast1`（東京）

### 5. Firestore Rules/Indexes デプロイ

```bash
cd limimeshi-admin

# .firebaserc に本番プロジェクトを追加
firebase use --add
# エイリアス名: prod
# プロジェクト: limimeshi-prod

# 本番環境にデプロイ
firebase use prod
firebase deploy --only firestore
```

### 6. 管理者 Custom Claims 設定

```bash
# 管理者UID を取得（Firebase Console → Authentication → Users）
node scripts/set-admin-claim.js <ADMIN_UID>
```

**スクリプト例** (`scripts/set-admin-claim.js`):

```javascript
const admin = require('firebase-admin');

// サービスアカウントキーを使用
const serviceAccount = require('./serviceAccountKey.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

const uid = process.argv[2];
if (!uid) {
  console.error('Usage: node set-admin-claim.js <UID>');
  process.exit(1);
}

admin.auth().setCustomUserClaims(uid, { admin: true })
  .then(() => {
    console.log(`Successfully set admin claim for ${uid}`);
    process.exit(0);
  })
  .catch((error) => {
    console.error('Error:', error);
    process.exit(1);
  });
```

### 7. Hosting 設定・デプロイ

```bash
# ビルド
npm run build

# デプロイ
firebase use prod
firebase deploy --only hosting:admin
```

### 8. Android アプリ設定

1. Firebase Console → プロジェクト設定 → マイアプリ
2. 「アプリを追加」→ Android
3. パッケージ名: `com.limimeshi.android`
4. `google-services.json` をダウンロード
5. `limimeshi-android/app/` に配置

---

## 方法B: Terraform による自動化（IaC）

### ディレクトリ構成

```
limimeshi-infra/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── environments/
│   │   ├── dev.tfvars
│   │   └── prod.tfvars
│   └── modules/
│       └── firebase/
│           ├── main.tf
│           └── variables.tf
├── scripts/
│   └── set-admin-claim.js
└── README.md
```

### Terraform 設定

**main.tf**:

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

provider "google-beta" {
  project = var.project_id
  region  = var.region
}

# Firebase プロジェクト有効化
resource "google_firebase_project" "default" {
  provider = google-beta
  project  = var.project_id
}

# Firestore データベース
resource "google_firestore_database" "default" {
  provider    = google-beta
  project     = var.project_id
  name        = "(default)"
  location_id = var.region
  type        = "FIRESTORE_NATIVE"

  depends_on = [google_firebase_project.default]
}

# Firebase Authentication（Identity Platform）
resource "google_identity_platform_config" "default" {
  provider = google-beta
  project  = var.project_id

  sign_in {
    allow_duplicate_emails = false

    email {
      enabled           = true
      password_required = true
    }
  }

  depends_on = [google_firebase_project.default]
}
```

**variables.tf**:

```hcl
variable "project_id" {
  description = "Firebase project ID"
  type        = string
}

variable "region" {
  description = "Default region"
  type        = string
  default     = "asia-northeast1"
}
```

**environments/prod.tfvars**:

```hcl
project_id = "limimeshi-prod"
region     = "asia-northeast1"
```

### Terraform 実行手順

```bash
cd limimeshi-infra/terraform

# 初期化
terraform init

# プラン確認
terraform plan -var-file=environments/prod.tfvars

# 適用
terraform apply -var-file=environments/prod.tfvars
```

### 制限事項（手動対応が必要）

| 項目 | 理由 |
|------|------|
| Blazeプラン切り替え | Terraform 非対応（課金設定のため） |
| Android アプリ登録 | Terraform 非対応（google-services.json生成のため） |
| 管理者ユーザー作成 | Firebase Console で作成後、スクリプトでClaims設定 |

---

## 環境切り替え

### Firebase CLI

```bash
# 開発環境
firebase use dev

# 本番環境
firebase use prod

# 現在の環境確認
firebase use
```

### limimeshi-admin 環境変数

**.env.local**（開発）:
```
VITE_FIREBASE_PROJECT_ID=limimeshi-dev
VITE_FIREBASE_API_KEY=xxx
...
```

**.env.production**（本番）:
```
VITE_FIREBASE_PROJECT_ID=limimeshi-prod
VITE_FIREBASE_API_KEY=xxx
...
```

### limimeshi-android 環境切り替え

`app/build.gradle.kts`:

```kotlin
android {
    buildTypes {
        debug {
            // limimeshi-dev を使用
        }
        release {
            // limimeshi-prod を使用
        }
    }
}
```

`google-services.json` は環境ごとに配置:
- `app/src/debug/google-services.json`（開発）
- `app/src/release/google-services.json`（本番）

---

## チェックリスト

### 本番環境セットアップ完了確認

- [ ] Firebase プロジェクト `limimeshi-prod` 作成
- [ ] Blazeプラン有効化
- [ ] Authentication（メール/パスワード）有効化
- [ ] Firestore データベース作成（asia-northeast1）
- [ ] Firestore Rules デプロイ
- [ ] Firestore Indexes デプロイ
- [ ] 管理者ユーザー作成
- [ ] 管理者 Custom Claims 設定
- [ ] limimeshi-admin Hosting デプロイ
- [ ] Android アプリ登録
- [ ] `google-services.json` 取得・配置

---

## 参考リンク

- [Firebase Console](https://console.firebase.google.com/)
- [Firebase CLI リファレンス](https://firebase.google.com/docs/cli)
- [Terraform Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Terraform Firebase リソース](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/firebase_project)
