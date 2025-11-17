# ADR-005: Deploy using Firebase Hosting multi-site

## Status

Accepted

## Context

期間限定めし（リミメシ）は、以下の複数のWebアプリケーションで構成される：

1. **管理画面（limimeshi-admin）**: チェーン店・メニューの登録・編集
2. **一般向けWebアプリ（limimeshi-web）**: メニュー一覧・お気に入り登録
3. **定期処理（limimeshi-jobs）**: Cloud Functions（スケジューラ）

これらのアプリケーションは、共通のFirestoreデータベースを使用する。

### 検討すべき課題

1. **Hosting構成**: 管理画面と一般向けWebアプリをどのように配置するか
2. **環境分離**: 開発環境と本番環境をどう分けるか
3. **共有リソース管理**: Firestore Rules/Indexesをどこで管理するか
4. **デプロイ戦略**: 各アプリケーションのデプロイフロー

### 検討した選択肢

#### Hosting構成

**Option 1: 同じFirebase Project、複数のHostingサイト（Multi-Site）**
- メリット: Firestoreを共有、管理が簡単、コスト効率
- デメリット: 管理画面と一般向けが同じProjectに同居

**Option 2: 別々のFirebase Project**
- メリット: 完全分離、セキュリティ
- デメリット: Firestoreが別々（データ同期が必要）、管理コスト高

**Option 3: 管理画面は別サービス（Vercel等）**
- メリット: Firebase Hostingの制約を受けない
- デメリット: 管理が複雑、Firestoreへの外部アクセス

#### 環境分離

**Option A: 環境ごとに別Firebase Project**
- メリット: 各環境が独自のリソースを持つ（Firebase公式推奨）
- デメリット: Projectが増える

**Option B: Multi-Site で環境を分ける**
- メリット: Projectが1つで済む
- デメリット: Firebase公式非推奨（環境ごとに別Projectを推奨）

#### Firestore Rules/Indexes 配置

**Option α: limimeshi-admin に配置**
- メリット: Phase2ではシンプル、管理画面が最も使う
- デメリット: limimeshi-web も使う場合、別リポジトリに設定がある

**Option β: limimeshi-infra（新規リポジトリ）に配置**
- メリット: インフラ設定を一元管理、各リポジトリの責務明確
- デメリット: リポジトリが増える

**Option γ: limimeshi-jobs に統合**
- メリット: Cloud Functions と関連
- デメリット: "jobs" という名前と役割が合わない

## Decision

### 1. Hosting構成: Multi-Site（Option 1）

**同じFirebase Project内で、複数のHostingサイトを使用**

- 管理画面と一般向けWebアプリを同一Firebase Projectに配置
- Firebase Hosting multi-site 機能を使用
- Firestoreデータベースを共有

**理由**:
- Phase2 MVPではシンプルさ優先
- Firestoreを共有できる（管理画面でデータ登録、一般向けで表示）
- Firebase Hosting multi-site は標準機能で安定稼働
- コスト効率が良い

### 2. 環境分離: 環境ごとに別Firebase Project（Option A）

**開発環境と本番環境で別々のFirebase Projectを作成**

Firebase公式ドキュメントの推奨に従う：
> 環境（Dev, Prod）のミラーリングには、別々のFirebase Projectを使うことを推奨。各環境が独自のFirebaseリソースを持つべき。

**構成**:

開発環境:
```
Firebase Project: limimeshi-dev
├── Firestore（開発用データ）
├── Authentication（開発用ユーザー）
├── Cloud Functions（開発用）
├── Hosting Site 1: limimeshi-admin-dev
│   └── URL: limimeshi-admin-dev.web.app
└── Hosting Site 2: limimeshi-web-dev
    └── URL: limimeshi-web-dev.web.app
```

本番環境:
```
Firebase Project: limimeshi-prod
├── Firestore（本番データ）
├── Authentication（本番ユーザー）
├── Cloud Functions（本番）
├── Hosting Site 1: limimeshi-admin
│   └── URL: limimeshi-admin.web.app
│   └── Custom Domain: admin.limimeshi.com（Phase3以降）
└── Hosting Site 2: limimeshi-web
    └── URL: limimeshi-web.web.app
    └── Custom Domain: limimeshi.com（Phase3以降）
```

**理由**:
- Firebase公式推奨のベストプラクティス
- 開発環境の障害が本番環境に影響しない
- 各環境で独立したデータ管理

### 3. Firestore Rules/Indexes 配置: 段階的移行（Option α → β）

**Phase2（暫定）: limimeshi-admin に配置**

```
limimeshi-admin/
├── src/                        # 管理画面
├── firestore.rules             # ★暫定的に同居
├── firestore.indexes.json      # ★暫定的に同居
├── firebase.json
└── .firebaserc
```

**理由**:
- Phase2では limimeshi-web が未実装
- すぐに開発開始できる
- シンプル

**Phase3（移行）: limimeshi-infra に分離**

移行タイミング: limimeshi-web リポジトリ作成の直前

```
limimeshi-infra/              # ★新設
├── firestore.rules
├── firestore.indexes.json
├── scripts/
│   ├── deploy-dev.sh
│   └── deploy-prod.sh
├── firebase.json
├── .firebaserc
└── README.md
```

**理由**:
- 共有リソースの一元管理
- 各リポジトリの責務明確化（admin/web はアプリロジックに専念）
- limimeshi-web が Rules を参照する前に分離することで、二重管理を回避

## Consequences

### メリット

**シンプルなPhase2開始**:
- limimeshi-admin のみで開発開始可能
- 追加リポジトリ不要

**明確な移行パス**:
- Phase3で limimeshi-infra に移行するタイミングが明確
- limimeshi-web 作成前に移行することで、二重管理を回避

**環境分離**:
- Dev/Prod で別Firebase Project
- 開発環境の障害が本番環境に影響しない

**Multi-Site の利点**:
- Firestoreを共有（管理画面でデータ登録、一般向けで表示）
- 管理が簡単（1つのFirebase Project）

**スケーラビリティ**:
- 将来的に limimeshi-infra でインフラ設定を一元管理
- 各リポジトリの責務が明確

### デメリットと対策

**Phase3で移行作業が必要**:
- 制約: limimeshi-infra 作成、firestore.rules/indexes の移動、デプロイスクリプト更新
- 対策: 移行手順を明確化（後述のMigration Plan参照）

**Blazeプラン必須**:
- 制約: Multi-Site 機能を使うため従量課金プラン必須
- 対策: 無料枠あり（Hosting: 10GB/月、転送: 360MB/日）、Phase2は無料枠内で運用可能

**一時的な二重配置**:
- 制約: Phase2では limimeshi-admin に firestore.rules/indexes、Phase3では limimeshi-infra に移行
- 対策: 移行期間は短く保つ（limimeshi-web 作成の直前に移行）

### リポジトリ構成

```
limimeshi-admin/              # 管理画面（hosting:admin のみ）
├── src/
├── firebase.json            # hosting 設定のみ
└── .firebaserc

limimeshi-web/                # 一般向けWebアプリ（hosting:web のみ）
├── src/
├── firebase.json            # hosting 設定のみ
└── .firebaserc

limimeshi-infra/              # インフラ設定（Phase3以降）
├── firestore.rules
├── firestore.indexes.json
├── firebase.json            # firestore 設定のみ
└── .firebaserc

limimeshi-jobs/               # 定期処理（Cloud Functions）
└── functions/
    └── src/
```

### デプロイ戦略

**Phase2（limimeshi-admin のみ）**:

開発環境デプロイ:
```bash
cd limimeshi-admin
firebase use dev
firebase deploy --only hosting:admin,firestore
```

内訳:
- `hosting:admin`: 管理画面の静的ファイル
- `firestore`: Firestore Rules/Indexes

**Phase3以降（limimeshi-infra 分離後）**:

Firestore Rules/Indexes 更新時:
```bash
cd limimeshi-infra
firebase use dev
firebase deploy --only firestore
```

管理画面更新時:
```bash
cd limimeshi-admin
firebase use dev
firebase deploy --only hosting:admin
```

一般向けWebアプリ更新時:
```bash
cd limimeshi-web
firebase use dev
firebase deploy --only hosting:web
```

### カスタムドメイン設定（Phase3以降）

ドメイン構成:
- **limimeshi.com**: 一般向けWebアプリ（本番）
- **admin.limimeshi.com**: 管理画面（本番）

DNS設定例（お名前.comなど）:
```
Type: CNAME
Host: admin
Value: limimeshi-admin.web.app

Type: CNAME
Host: @（または省略）
Value: limimeshi-web.web.app
```

**Phase2**: カスタムドメイン未設定、デフォルトURL（*.web.app）のみ使用

---

## 補足: Migration Plan（Phase3移行手順）

### タイミング

limimeshi-web リポジトリを `git init` する直前

### 手順

1. **limimeshi-infra リポジトリ作成**
   ```bash
   mkdir limimeshi-infra
   cd limimeshi-infra
   git init
   ```

2. **Firestore Rules/Indexes を移行**
   ```bash
   # limimeshi-admin から移動
   cp limimeshi-admin/firestore.rules limimeshi-infra/
   cp limimeshi-admin/firestore.indexes.json limimeshi-infra/
   ```

3. **firebase.json 作成（limimeshi-infra）**
   ```json
   {
     "firestore": {
       "rules": "firestore.rules",
       "indexes": "firestore.indexes.json"
     }
   }
   ```

4. **.firebaserc 作成（limimeshi-infra）**
   ```json
   {
     "projects": {
       "dev": "limimeshi-dev",
       "prod": "limimeshi-prod"
     }
   }
   ```

5. **limimeshi-admin から firestore 設定を削除**
   ```json
   {
     "hosting": {
       "target": "admin",
       "public": "dist",
       ...
     }
     // "firestore" セクションを削除
   }
   ```

6. **デプロイスクリプト作成（limimeshi-infra）**
   ```bash
   # scripts/deploy-dev.sh
   firebase use dev && firebase deploy --only firestore

   # scripts/deploy-prod.sh
   firebase use prod && firebase deploy --only firestore
   ```

7. **動作確認**
   ```bash
   cd limimeshi-infra
   bash scripts/deploy-dev.sh
   ```

8. **limimeshi-admin から削除**
   ```bash
   cd limimeshi-admin
   rm firestore.rules firestore.indexes.json
   git commit -m "refactor: Firestore Rules/Indexesをlimimeshi-infraに移行"
   ```

9. **limimeshi-web 作成開始**
   firestore.rules/indexes は limimeshi-infra を参照

---

## 補足: 参考リンク

- [Firebase Hosting - Multiple Sites](https://firebase.google.com/docs/hosting/multisites)
- [Firebase Hosting - Custom Domain](https://firebase.google.com/docs/hosting/custom-domain)
- [Firebase - Multi-Project Setup](https://firebase.google.com/docs/projects/manage-installations#projects-and-apps)
