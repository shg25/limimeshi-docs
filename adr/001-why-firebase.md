# ADR-001: なぜFirebaseを選んだか

**Status**: 承認済み
**Date**: 2025-11-16
**Author**: 重次弘規
**Related**: [memory/constitution.md](../memory/constitution.md), [planning/first-idea.md](../planning/first-idea.md)

## Context

期間限定めし（リミメシ）は、以下の要件を満たすバックエンド・インフラを必要とする：

### プロジェクト特性
- **個人開発**：開発者1名、運用も1名体制
- **予算制約**：月0〜5,000円（Phase2）、Phase3以降も低コスト運用
- **開発期間**：Phase2を早期にリリースしたい（MVP重視）
- **スケーラビリティ**：将来的に月間1,000人〜10,000人のアクティブユーザーを想定

### 技術要件
- **認証**：Googleログイン、匿名認証、管理者認証（Custom Claims）
- **データベース**：チェーン店・メニュー・お気に入りデータの管理
- **ホスティング**：一般向けWebアプリ、管理画面の配信
- **サーバーレス関数**：定期処理（将来の差分検知等）
- **セキュリティ**：Firestoreセキュリティルール、認証必須機能

### 検討した選択肢

#### Option 1: Firebase（採用）
- **認証**: Firebase Authentication
- **データベース**: Firestore（NoSQL）
- **ホスティング**: Firebase Hosting
- **サーバーレス**: Cloud Functions
- **その他**: Cloud Scheduler, Firebase Analytics

#### Option 2: Supabase
- **認証**: Supabase Auth（PostgreSQL）
- **データベース**: PostgreSQL（RDB）
- **ホスティング**: Vercel等の別サービス
- **サーバーレス**: Edge Functions（Deno）

#### Option 3: AWS（Amplify）
- **認証**: Cognito
- **データベース**: DynamoDB / RDS
- **ホスティング**: S3 + CloudFront
- **サーバーレス**: Lambda

#### Option 4: 完全自作（フルスタック）
- **認証**: Auth0 / Clerk
- **データベース**: MongoDB Atlas / PostgreSQL（RDS）
- **ホスティング**: Vercel / Netlify
- **サーバーレス**: Vercel Functions / Netlify Functions

## Decision

**Firebase** を採用する

### 選定理由

#### 1. 個人開発での運用効率（最重要）

**Firebase**:
- ✅ バックエンドサーバーの管理不要
- ✅ 認証・DB・ホスティング・Functionsがオールインワン
- ✅ 管理画面（Firebase Console）で一元管理
- ✅ デプロイが簡単（`firebase deploy`）

**Supabase**:
- ❌ ホスティングは別サービス（Vercel等）が必要
- ❌ Edge Functionsが Deno（学習コスト）

**AWS Amplify**:
- ❌ 管理画面が複雑（IAM、VPC、セキュリティグループ等）
- ❌ 学習コストが高い

**完全自作**:
- ❌ サーバー管理・セキュリティ対策が必要
- ❌ 認証・DB・ホスティングを個別に管理（運用負荷高）

#### 2. コスト管理

**Firebase**:
- ✅ 無料枠が充実（Authentication: 10,000認証/月、Firestore: 50,000読み/1日）
- ✅ 予算アラート設定が簡単
- ✅ 従量課金で予測可能

**Supabase**:
- ✅ 無料プラン有り（PostgreSQL 500MB）
- ❌ ホスティングは別料金

**AWS Amplify**:
- ❌ 無料枠が限定的（Lambda 100万リクエスト/月）
- ❌ 料金体系が複雑

**完全自作**:
- ❌ 各サービスの料金を個別管理（コスト予測困難）

#### 3. 開発速度

**Firebase**:
- ✅ React Admin との統合が容易（react-admin-firebase）
- ✅ ドキュメントが充実
- ✅ プロトタイプ作成が最速

**Supabase**:
- ✅ PostgreSQLで型安全
- ❌ React Adminとの統合ライブラリが少ない

**AWS Amplify**:
- ❌ セットアップが複雑

**完全自作**:
- ❌ 開発期間が長い

#### 4. セキュリティ

**Firebase**:
- ✅ Googleのセキュリティインフラ
- ✅ Firestoreセキュリティルールで細かい制御
- ✅ Custom Claims で権限管理

**Supabase**:
- ✅ Row Level Security（RLS）で細かい制御
- ✅ PostgreSQLの成熟したセキュリティ

**AWS Amplify**:
- ✅ AWSのセキュリティインフラ
- ❌ IAM設定が複雑

**完全自作**:
- ❌ セキュリティ対策を自前実装（高リスク）

#### 5. スケーラビリティ

**Firebase**:
- ✅ 自動スケーリング
- ✅ 1,000万ユーザーまで対応可能（実績あり）

**Supabase**:
- ✅ PostgreSQLの垂直スケーリング
- ❌ 無料プランは制限あり

**AWS Amplify**:
- ✅ 無制限スケーリング
- ❌ コストが急増

**完全自作**:
- ❌ スケーリング対応を自前実装

#### 6. エコシステム

**Firebase**:
- ✅ React Admin統合（react-admin-firebase）
- ✅ 豊富なライブラリ・コミュニティ

**Supabase**:
- ✅ 急成長中のコミュニティ
- ❌ React Adminとの統合が弱い

**AWS Amplify**:
- ✅ 豊富なライブラリ
- ❌ 学習コスト高

**完全自作**:
- ❌ 各サービスの統合を自前実装

### 比較表

| 評価軸 | Firebase | Supabase | AWS Amplify | 完全自作 |
|--------|----------|----------|-------------|----------|
| **運用効率** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐ |
| **コスト** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **開発速度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **セキュリティ** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **スケーラビリティ** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **React Admin統合** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐ |
| **学習コスト** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ |

### Phase2で最も重要な評価軸

1. **運用効率**：個人開発、1名運用 → Firebase が最適
2. **開発速度**：早期リリース重視 → Firebase が最適
3. **コスト**：月0〜5,000円 → Firebase が最適

## Consequences

### メリット

#### 運用効率
- バックエンドサーバーの管理不要
- Firebase Consoleで全てのリソースを一元管理
- デプロイが簡単（`firebase deploy`）

#### 開発速度
- React Admin との統合が容易（react-admin-firebase）
- 認証・DB・ホスティング・Functionsがオールインワン
- プロトタイプ作成が最速

#### コスト
- 無料枠が充実（Phase2は無料枠内で運用可能）
- 予算アラート設定で予算爆発を防止
- 従量課金で予測可能

#### セキュリティ
- Googleのセキュリティインフラ
- Firestoreセキュリティルールで細かい制御
- Custom Claims で管理者権限管理

#### スケーラビリティ
- 自動スケーリング
- 1,000万ユーザーまで対応可能

### デメリットと対策

#### RDBが使えない
- **制約**: Firestoreは NoSQL、JOIN・トランザクションが弱い
- **対策**: データモデル設計で非正規化（お気に入り登録数の集約等）

#### ベンダーロックイン
- **制約**: Firebaseに依存、他のサービスへの移行コスト高
- **対策**: Phase2ではFirebaseで実装、Phase3以降で再評価（必要に応じて移行）

#### Firestore料金の急増リスク
- **制約**: 読み書き回数が増えるとコスト急増
- **対策**: インデックス設計の最適化、予算アラート設定、負荷テスト実施

#### Custom Claimsの制約
- **制約**: Custom Claimsは1KB以内、複雑な権限管理に不向き
- **対策**: Phase2は管理者のみ（`admin: true`）、Phase3以降でRBACを検討

### 実装時の注意点

#### Firestore Security Rules
- 管理者認証（Custom Claims）を使った権限チェック
- 一般ユーザーと管理者で明確に分岐
- `isAdmin()` / `isOwner()` を共通関数化

#### Firestoreインデックス
- クエリごとにインデックスを設計
- 複合インデックスの作成（`chainId` + `saleStartTime`）

#### Cloud Functions
- メモリ使用量・実行回数の監視
- コールドスタート対策（最小メモリ256MB）

#### 予算管理
- Firebase予算上限の設定（月0〜5,000円）
- 予算アラートの設定（80%到達時に通知）

### 将来の見直しタイミング

以下の状況になった場合、再評価する：
- Firestoreの読み書き回数が無料枠を超え、コストが月5,000円を超過
- RDBが必要になる（トランザクション・JOIN多用）
- AWS Amplifyの学習を終え、移行コストが許容範囲

## Notes

- Constitution IV. Firebase-First に従う
- ADR-002（マルチリポ選定）、ADR-003（React Admin選定）、ADR-005（Firebase Hosting構成）の前提条件
- Phase2ではFirebaseのみ使用、他のサービスは検討しない
