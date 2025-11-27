# ADR-002: Adopt multi-repository structure

## Status

Accepted

## Context

期間限定めし（リミメシ）は、以下の複数のアプリケーションで構成される：

1. **limimeshi-docs**: 企画・設計・ADR・仕様書（公開リポジトリ）
2. **limimeshi-admin**: 管理画面（React Admin）
3. **limimeshi-android**: 一般向けAndroidアプリ（Kotlin + Jetpack Compose）
4. **limimeshi-jobs**: 定期処理・差分検知（Cloud Functions）
5. **（将来）limimeshi-api**: 共通API / BFF（Cloud Functions）
6. **（将来）limimeshi-ios**: iOSアプリ

これらのアプリケーションは、共通のFirestoreデータベースを使用する。

### リポジトリ構成の選択肢

**Option 1: モノレポ（Monorepo）**

全てのアプリケーションを1つのリポジトリで管理

典型的な構造:
```
limimeshi/
├── packages/
│   ├── admin/
│   ├── web/
│   ├── jobs/
│   └── shared/  # 共通コード
├── docs/
└── package.json  # ルート package.json
```

ツール: Turborepo, Nx, Lerna, pnpm workspaces

**Option 2: マルチリポ（Multirepo）**

各アプリケーションを別々のリポジトリで管理

典型的な構造:
```
limimeshi-docs/       # 公開リポジトリ
limimeshi-admin/      # 公開リポジトリ
limimeshi-android/    # 公開リポジトリ
limimeshi-jobs/       # 公開リポジトリ
limimeshi-shared/     # 共通ライブラリ（公開リポジトリ）
```

### 検討すべき観点

1. **コード共有**: 共通型定義・ユーティリティの再利用
2. **デプロイ**: 各アプリケーションの独立したデプロイ
3. **CI/CD**: ビルド・テストの並列実行
4. **リリース管理**: バージョン管理・リリースノート
5. **開発効率**: 開発者1名での運用効率

### 評価軸と比較

| 評価軸 | マルチリポ | モノレポ |
|--------|----------|----------|
| **デプロイ独立性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **CI/CD単純性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **個人開発運用** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **共通コード管理** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **リリース管理** | ⭐⭐⭐⭐ | ⭐⭐⭐ |

## Decision

**マルチリポ（Multirepo）** を採用する

### 選定理由

#### 1. デプロイの独立性（最重要）

- 各アプリケーションを独立してデプロイ
- limimeshi-admin の変更が limimeshi-android に影響しない
- リリース管理が簡単（各リポジトリで独立したバージョン）

#### 2. CI/CDの単純化

- 各リポジトリで独立した GitHub Actions
- ビルド失敗が他のアプリに影響しない
- CI/CD設定がシンプル

#### 3. 個人開発での運用効率

- リポジトリごとに集中できる（コンテキストスイッチが少ない）
- 各リポジトリの責務が明確
- Git操作がシンプル

#### 4. Phase2での判断

Phase2（MVP）では以下の特性がマルチリポに有利：
- デプロイの独立性: 管理画面と一般向けAndroidアプリを独立してデプロイ
- CI/CDの単純化: 各リポジトリで独立したGitHub Actions
- 個人開発: 1名運用、リポジトリごとに集中できる方が効率的

## Consequences

### メリット

**デプロイの独立性**:
- 各アプリケーションを独立してデプロイ
- limimeshi-admin の変更が limimeshi-android に影響しない
- リリース管理が簡単

**CI/CDの単純化**:
- 各リポジトリで独立した GitHub Actions
- ビルド失敗が他のアプリに影響しない
- CI/CD設定がシンプル

**個人開発での運用効率**:
- リポジトリごとに集中できる
- 各リポジトリの責務が明確
- Git操作がシンプル

### デメリットと対策

**共通コードの管理**:
- 制約: 共通型定義・ユーティリティを各リポジトリで重複管理
- 対策（Phase2）: 共通コードは最小限に抑える（Firestoreスキーマの型定義等）
- 対策（Phase3）: limimeshi-shared リポジトリを作成、npm private package として公開

**バージョン管理の複雑性**:
- 制約: 各リポジトリで独立したバージョン管理
- 対策: セマンティックバージョニングを採用（v1.0.0, v1.1.0等）

**リポジトリ間の依存関係**:
- 制約: limimeshi-android が limimeshi-jobs の更新に依存する場合がある
- 対策: Firestoreスキーマを明確に定義、破壊的変更は慎重に実施

**公開リポジトリのセキュリティ管理**:
- 制約: ソースコードが公開されるため、セキュリティ上の注意が必要
- 対策: `.env` ファイル等の秘匿情報は `.gitignore` で除外
- 対策: Firestore Security Rulesで適切にセキュリティを保護
- 対策: Firebase APIキーは公開されても問題ない（Firestore Security Rulesで保護）
- 対策: サービスアカウントキー（`.json`）は絶対にコミットしない

### Phase2での実装方針

**リポジトリ構成**:
```
limimeshi-docs/       # 公開リポジトリ
├── planning/
├── specs/
├── adr/
└── README.md

limimeshi-admin/      # 公開リポジトリ
├── src/
├── firestore.rules   # Phase2暫定配置
├── firestore.indexes.json
├── .env.example      # 環境変数のテンプレート
├── .gitignore        # .env を除外
└── package.json
```

**共通コードの扱い（Phase2）**:
- Phase2では共通ライブラリを作成しない
- Firestoreスキーマの型定義は各リポジトリで定義
- 重複を許容（Phase2は管理画面のみ、重複コード量が少ない）

**デプロイフロー**:
各リポジトリで独立した GitHub Actions を設定し、`firebase deploy` で各アプリケーションをデプロイ

### 将来の見直しタイミング

以下の状況になった場合、モノレポを再検討する：
- 共通コード量が増え、管理コストが高くなった場合
- リポジトリ間の依存関係が複雑になった場合
- チーム開発に移行し、全体のコードを一元管理したい場合
