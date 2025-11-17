# ADR-001: Use Firebase for backend

## Status

Accepted

## Context

期間限定めし（リミメシ）は、以下の要件を満たすバックエンド・インフラを必要とする：

### プロジェクト特性
- **個人開発**: 開発者1名、運用も1名体制
- **予算制約**: 月0〜5,000円（Phase2）、Phase3以降も低コスト運用
- **開発期間**: Phase2を早期にリリースしたい（MVP重視）
- **スケーラビリティ**: 将来的に月間1,000人〜10,000人のアクティブユーザーを想定
- **モバイルアプリ展開**: Phase5以降でiOS/Androidネイティブアプリを開発予定（学習目的含む）

### 技術要件
- **認証**: Googleログイン、匿名認証、管理者認証（Custom Claims）
- **データベース**: チェーン店・メニュー・お気に入りデータの管理
- **ホスティング**: 一般向けWebアプリ、管理画面の配信
- **サーバーレス関数**: 定期処理（将来の差分検知等）
- **セキュリティ**: Firestoreセキュリティルール、認証必須機能
- **モバイルアプリ統合**: WebアプリとiOS/Androidアプリで認証・データベースを共有

### 検討した選択肢

**Option 1: Firebase**
- 認証: Firebase Authentication
- データベース: Firestore（NoSQL）
- ホスティング: Firebase Hosting
- サーバーレス: Cloud Functions

**Option 2: Supabase**
- 認証: Supabase Auth（PostgreSQL）
- データベース: PostgreSQL（RDB）
- ホスティング: Vercel等の別サービス
- サーバーレス: Edge Functions（Deno）

**Option 3: AWS（Amplify）**
- 認証: Cognito
- データベース: DynamoDB / RDS
- ホスティング: S3 + CloudFront
- サーバーレス: Lambda

**Option 4: 完全自作（フルスタック）**
- 認証: Auth0 / Clerk
- データベース: MongoDB Atlas / PostgreSQL（RDS）
- ホスティング: Vercel / Netlify
- サーバーレス: Vercel Functions / Netlify Functions

### 評価軸と比較

| 評価軸 | Firebase | Supabase | AWS Amplify | 完全自作 |
|--------|----------|----------|-------------|----------|
| **運用効率** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐ |
| **コスト** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **開発速度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **セキュリティ** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **スケーラビリティ** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **React Admin統合** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐ |
| **モバイルアプリ統合** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

**Phase2で最も重要な評価軸**:
1. **運用効率**: 個人開発、1名運用 → Firebase が最適
2. **開発速度**: 早期リリース重視 → Firebase が最適
3. **コスト**: 月0〜5,000円 → Firebase が最適
4. **モバイルアプリ統合**: Phase5以降のiOS/Android展開を見据える → Firebase が最適

## Decision

**Firebase** を採用する

### 選定理由

#### 1. 個人開発での運用効率（最重要）

- バックエンドサーバーの管理不要
- 認証・DB・ホスティング・Functionsがオールインワン
- Firebase Consoleで一元管理
- デプロイが簡単（`firebase deploy`）

#### 2. コスト管理

- 無料枠が充実（Authentication: 10,000認証/月、Firestore: 50,000読み/1日）
- 予算アラート設定が簡単
- 従量課金で予測可能
- Phase2は無料枠内で運用可能

#### 3. 開発速度

- React Admin との統合が容易（react-admin-firebase）
- ドキュメントが充実
- プロトタイプ作成が最速

#### 4. セキュリティ

- Googleのセキュリティインフラ
- Firestoreセキュリティルールで細かい制御
- Custom Claims で管理者権限管理

#### 5. スケーラビリティ

- 自動スケーリング
- 1,000万ユーザーまで対応可能（実績あり）

#### 6. iOS/Androidアプリとの統合

- Firebase SDK for iOS/Android が成熟（SwiftUI、Jetpack Compose対応）
- WebアプリとモバイルアプリでFirestoreデータベースを共有
- Firebase Authenticationで認証を統一（匿名認証→Googleログイン、UID継続）
- Phase5以降のモバイルアプリ展開を見据えた選定
- React Native（Expo）にも対応（クロスプラットフォーム開発も可能）
- **モバイルアプリ向けFirebase機能**:
  - Crashlytics: クラッシュレポート
  - App Distribution: テスト配布（TestFlight/Google Play Consoleの代替）
  - Google Analytics: アプリ分析（GA4統合）
  - Cloud Messaging: プッシュ通知（APNs/FCM統合）

## Consequences

### メリット

**運用効率**:
- バックエンドサーバーの管理不要
- Firebase Consoleで全てのリソースを一元管理
- デプロイが簡単（`firebase deploy`）

**開発速度**:
- React Admin との統合が容易（react-admin-firebase）
- 認証・DB・ホスティング・Functionsがオールインワン
- プロトタイプ作成が最速

**コスト**:
- 無料枠が充実（Phase2は無料枠内で運用可能）
- 予算アラート設定で予算爆発を防止
- 従量課金で予測可能

**セキュリティ**:
- Googleのセキュリティインフラ
- Firestoreセキュリティルールで細かい制御
- Custom Claims で管理者権限管理

**スケーラビリティ**:
- 自動スケーリング
- 1,000万ユーザーまで対応可能

**モバイルアプリ統合**:
- Firebase SDK for iOS/Android が成熟（SwiftUI、Jetpack Compose対応）
- WebアプリとモバイルアプリでFirestoreデータベースを共有
- Firebase Authenticationで認証を統一（匿名認証→Googleログイン、UID継続）
- Phase5以降のモバイルアプリ展開を見据えた選定
- React Native（Expo）にも対応（クロスプラットフォーム開発も可能）
- モバイルアプリ向けFirebase機能（Crashlytics、App Distribution、Google Analytics、Cloud Messaging）がオールインワン

### デメリットと対策

**RDBが使えない**:
- 制約: Firestoreは NoSQL、JOIN・トランザクションが弱い
- 対策: データモデル設計で非正規化（お気に入り登録数の集約等）

**ベンダーロックイン**:
- 制約: Firebaseに依存、他のサービスへの移行コスト高
- 対策: Phase2ではFirebaseで実装、Phase3以降で再評価（必要に応じて移行）

**Firestore料金の急増リスク**:
- 制約: 読み書き回数が増えるとコスト急増
- 対策: インデックス設計の最適化、予算アラート設定、負荷テスト実施

**Custom Claimsの制約**:
- 制約: Custom Claimsは1KB以内、複雑な権限管理に不向き
- 対策: Phase2は管理者のみ（`admin: true`）、Phase3以降でRBACを検討

### 実装時の注意点

**Firestore Security Rules**:
- 管理者認証（Custom Claims）を使った権限チェック
- 一般ユーザーと管理者で明確に分岐
- `isAdmin()` / `isOwner()` を共通関数化

**Firestoreインデックス**:
- クエリごとにインデックスを設計
- 複合インデックスの作成（`chainId` + `saleStartTime`）

**Cloud Functions**:
- メモリ使用量・実行回数の監視
- コールドスタート対策（最小メモリ256MB）

**予算管理**:
- Firebase予算上限の設定（月0〜5,000円）
- 予算アラートの設定（80%到達時に通知）

**Firebase Hosting vs Firebase App Hosting**:
- Phase2ではFirebase Hostingを使用（SPA: Single Page Application）
- React Admin（管理画面）とReact（一般向けWebアプリ）は静的ファイルとして配信され、ブラウザ上でFirestoreと通信
- データ編集はブラウザからFirestore APIを直接呼び出す（サーバー不要）
- Firebase App Hostingは使用しない（Next.js等のSSRフレームワーク向け）
- Phase3以降でSEO対策等でSSRが必要になった場合、Firebase App Hostingを検討

### 将来の見直しタイミング

以下の状況になった場合、再評価する：
- Firestoreの読み書き回数が無料枠を超え、コストが月5,000円を超過
- RDBが必要になる（トランザクション・JOIN多用）
- AWS Amplifyの学習を終え、移行コストが許容範囲
