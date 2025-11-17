# ADR-002: Adopt multi-repository structure

## Status

Accepted

## Context

期間限定めし（リミメシ）は、以下の複数のアプリケーションで構成される：

1. **limimeshi-docs**: 企画・設計・ADR・仕様書（公開リポジトリ）
2. **limimeshi-admin**: 管理画面（React Admin）
3. **limimeshi-web**: 一般向けWebアプリ（React）
4. **limimeshi-jobs**: 定期処理・差分検知（Cloud Functions）
5. **（将来）limimeshi-api**: 共通API / BFF（Cloud Functions）
6. **（将来）limimeshi-expo/ios/android**: モバイルアプリ

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
limimeshi-admin/      # プライベートリポジトリ
limimeshi-web/        # プライベートリポジトリ
limimeshi-jobs/       # プライベートリポジトリ
limimeshi-shared/     # 共通ライブラリ（プライベートリポジトリ）
```

### 検討すべき観点

1. **コード共有**: 共通型定義・ユーティリティの再利用
2. **デプロイ**: 各アプリケーションの独立したデプロイ
3. **CI/CD**: ビルド・テストの並列実行
4. **リリース管理**: バージョン管理・リリースノート
5. **公開・非公開**: ドキュメントは公開、実装は非公開
6. **開発効率**: 開発者1名での運用効率

### 評価軸と比較

| 評価軸 | マルチリポ | モノレポ |
|--------|----------|----------|
| **公開・非公開分離** | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **デプロイ独立性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **CI/CD単純性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **個人開発運用** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **共通コード管理** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **リリース管理** | ⭐⭐⭐⭐ | ⭐⭐⭐ |

## Decision

**マルチリポ（Multirepo）** を採用する

### 選定理由

#### 1. 公開・非公開の分離（最重要）

- limimeshi-docs は公開リポジトリ
- limimeshi-admin/web/jobs は非公開リポジトリ
- 企画・設計の透明性と、実装の保護を両立

モノレポでは公開リポジトリにすると実装コードも公開され、非公開リポジトリにするとドキュメントを公開できない。

#### 2. デプロイの独立性

- 各アプリケーションを独立してデプロイ
- limimeshi-admin の変更が limimeshi-web に影響しない
- リリース管理が簡単（各リポジトリで独立したバージョン）

#### 3. CI/CDの単純化

- 各リポジトリで独立した GitHub Actions
- ビルド失敗が他のアプリに影響しない
- CI/CD設定がシンプル

#### 4. 個人開発での運用効率

- リポジトリごとに集中できる（コンテキストスイッチが少ない）
- 各リポジトリの責務が明確
- Git操作がシンプル

#### 5. Phase2での判断

Phase2（MVP）では以下の特性がマルチリポに有利：
- 企画・設計の透明性: limimeshi-docs を公開リポジトリとして運用
- 実装の保護: limimeshi-admin/web を非公開リポジトリとして保護
- デプロイの独立性: 管理画面と一般向けWebアプリを独立してデプロイ
- 個人開発: 1名運用、リポジトリごとに集中できる方が効率的

## Consequences

### メリット

**公開・非公開の分離**:
- limimeshi-docs を公開リポジトリとして運用
- 企画・設計・ADRを透明性高く公開
- 実装コード（limimeshi-admin/web/jobs）は非公開

**デプロイの独立性**:
- 各アプリケーションを独立してデプロイ
- limimeshi-admin の変更が limimeshi-web に影響しない
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
- 制約: limimeshi-web が limimeshi-jobs の更新に依存する場合がある
- 対策: Firestoreスキーマを明確に定義、破壊的変更は慎重に実施

### Phase2での実装方針

**リポジトリ構成**:
```
limimeshi-docs/       # 公開リポジトリ
├── planning/
├── specs/
├── adr/
└── README.md

limimeshi-admin/      # プライベートリポジトリ
├── src/
├── firestore.rules   # Phase2暫定配置
├── firestore.indexes.json
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
