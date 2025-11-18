# specs/ - 機能設計書（GitHub Spec Kit形式）

このディレクトリには、各機能の設計ドキュメントが格納されています。

## ディレクトリ構成

```
specs/
├── README.md                    # このファイル
├── 001-admin-panel/             # 管理画面機能
│   ├── spec.md                  # 機能仕様書
│   ├── research.md              # 技術調査
│   ├── contracts/               # API契約・スキーマ定義
│   │   └── firestore-schema.md
│   ├── quickstart.md            # 開発環境構築・実装ガイド
│   ├── plan.md                  # 実装計画
│   ├── tasks.md                 # タスクリスト
│   └── checklists/              # チェックリスト
│       └── requirements.md
├── 002-menu-list/               # メニュー一覧機能
│   ├── spec.md
│   ├── research.md
│   ├── contracts/
│   │   └── firestore-schema.md
│   ├── quickstart.md
│   ├── plan.md
│   ├── tasks.md
│   └── checklists/
│       └── requirements.md
└── 003-favorites/               # お気に入り登録機能
    ├── spec.md
    ├── research.md
    ├── contracts/
    │   └── firestore-schema.md
    ├── quickstart.md
    ├── plan.md
    └── tasks.md
```

## GitHub Spec Kitとは

**GitHub Spec Kit**は、GitHubが提唱する仕様駆動開発（Spec-Driven Development）のフレームワーク

### 基本哲学

- **仕様が王様、コードは従者**: 仕様書がすべての開発の起点
- **AI補助前提**: Claude CodeなどのAIエージェントが仕様書からコードを生成
- **仕様と実装のギャップをゼロに**: 仕様書とコードを常に同期

### 公式リンク

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [公式ドキュメント](https://github.com/github/spec-kit/blob/main/README.md)

## ファイル生成順序（GitHub Spec Kit公式ワークフロー）

### Phase 0: 仕様書作成

**コマンド**: `/speckit-specify`

**生成されるファイル**:
- `spec.md`: 機能仕様書（ユーザーストーリー、要求仕様、成功基準）

**目的**: ユーザーの要求を明確化し、実装すべき機能を定義

**補助コマンド**: `/speckit-clarify`（仕様の曖昧な点を明確化）

---

### Phase 1: 実装計画作成

**コマンド**: `/speckit-plan`

**生成されるファイル**:
1. **research.md**: 技術調査
   - 技術選定の理由
   - 代替案の検討
   - Constitution（開発原則）への準拠確認
   - 各技術の使用例とベストプラクティス

2. **contracts/**: API契約・スキーマ定義
   - `firestore-schema.md`: Firestoreコレクション構造、Security Rules
   - `api-spec.md`: REST API仕様（該当機能のみ）
   - `types.md`: TypeScript型定義（該当機能のみ）

3. **quickstart.md**: 開発環境構築・実装ガイド
   - 前提条件（依存関係、環境設定）
   - コア実装例（最小限の動作サンプルコード）
   - テスト例（単体テスト、E2Eテストのサンプル）
   - トラブルシューティング（よくある問題と解決方法）

4. **plan.md**: 実装計画
   - Summary: 機能概要と技術アプローチ
   - Technical Context: 技術スタック、依存関係、制約条件
   - Constitution Check: 開発原則（9項目）への準拠確認
   - Project Structure: ソースコードとドキュメントの構造
   - Implementation Phases: Phase 0（Research）、Phase 1（Design & Contracts）の成果物
   - Dependencies: 前提条件、提供データ、他機能との依存関係
   - Risks & Mitigation: リスク分析と対策
   - Success Metrics: 成功基準（定量的指標）

**目的**: 技術選定、設計決定、実装戦略を文書化

---

### Phase 2: タスクリスト生成

**コマンド**: `/speckit-tasks`

**生成されるファイル**:
- `tasks.md`: タスクリスト
  - Phase 1: Setup（共有インフラストラクチャ）
  - Phase 2: Foundational（ブロッキング前提条件）
  - Phase 3+: User Stories（ユーザーストーリーごとのタスク）
  - Final Phase: Polish & Cross-Cutting Concerns
  - Dependencies & Execution Order（依存関係と実行順序）
  - Parallel Opportunities（並列実行可能なタスク）

**目的**: 実装タスクを細分化し、実行可能な単位に分割

**タスクフォーマット**:
```
- [ ] [T001] [P?] [US1?] Description with file path
```
- `[T001]`: タスクID
- `[P]`: 並列実行可能（Parallelizable）
- `[US1]`: ユーザーストーリー番号
- `Description with file path`: タスク内容と対象ファイルパス

---

### Phase 3: 実装

**コマンド**: `/speckit-implement`

**目的**: tasks.mdのタスクを順次実行し、コードを実装

**実装の流れ**:
1. テストを先に書く（TDD）
2. テストが失敗することを確認
3. 実装
4. テストが成功することを確認
5. コミット

**Note**: 実装フェーズでは、specs/ディレクトリではなく、実装リポジトリ（limimeshi-web、limimeshi-admin等）でコードを書く

---

## 各ファイルの説明

### spec.md（機能仕様書）

**Phase 0で生成**（`/speckit-specify`コマンド）

**内容**:
- User Scenarios & Testing: ユーザーストーリー（Priority付き）、Acceptance Scenarios
- Requirements: 機能要求（FR-001〜）、Key Entities
- Success Criteria: 成功基準（定量的指標）
- Assumptions: 前提条件
- Out of Scope: 対象外の機能

**目的**: ユーザーの要求を明確化し、実装すべき機能を定義

---

### research.md（技術調査）

**Phase 1で生成**（`/speckit-plan`コマンド）

**内容**:
- 各技術の選定理由（Decision）
- 選定根拠（Rationale）
- 代替案の検討（Alternatives Considered）
- 使用例（Key APIs / Key Pattern）
- 参考資料（Reference）

**目的**: 技術選定の理由を文書化し、後から振り返れるようにする

**例**:
- React + TypeScript（フロントエンド）
- Firestore SDK（データ読み取り）
- Firebase Authentication（ログイン状態の管理）
- Material-UI（UIコンポーネント）
- Vitest + Playwright（テスト戦略）

---

### contracts/（API契約・スキーマ定義）

**Phase 1で生成**（`/speckit-plan`コマンド）

**内容**:
- `firestore-schema.md`: Firestoreコレクション構造、フィールド定義、Security Rules、インデックス、パフォーマンス考慮
- `api-spec.md`: REST API仕様（該当機能のみ）
- `types.md`: TypeScript型定義（該当機能のみ）

**目的**: データ構造とAPI契約を明確化し、フロントエンド・バックエンド間の齟齬を防ぐ

---

### quickstart.md（開発環境構築・実装ガイド）

**Phase 1で生成**（`/speckit-plan`コマンド）

**内容**:
- Prerequisites: 前提条件（依存関係、環境設定）
- Setup: 環境構築手順
- Core Implementation: コア実装例（最小限の動作サンプルコード）
- Testing: テスト例（単体テスト、E2Eテストのサンプル）
- Deployment: デプロイ手順
- Troubleshooting: よくある問題と解決方法

**目的**: 新規参加者が素早く開発を開始できるようにする

---

### plan.md（実装計画）

**Phase 1で生成**（`/speckit-plan`コマンド）

**内容**:
- Summary: 機能概要と技術アプローチ
- Technical Context: 技術スタック、依存関係、制約条件
- Constitution Check: 開発原則（9項目）への準拠確認
- Project Structure: ソースコードとドキュメントの構造
- Complexity Tracking: 複雑性の追跡（Constitution違反の正当化）
- Implementation Phases: Phase 0（Research）、Phase 1（Design & Contracts）の成果物
- Dependencies: 前提条件、提供データ、他機能との依存関係
- Risks & Mitigation: リスク分析と対策
- Success Metrics: 成功基準（定量的指標）

**目的**: 実装計画を一元管理し、全体像を把握する

---

### tasks.md（タスクリスト）

**Phase 2で生成**（`/speckit-tasks`コマンド）

**内容**:
- Phase 1: Setup（共有インフラストラクチャ）
- Phase 2: Foundational（ブロッキング前提条件）
- Phase 3+: User Stories（ユーザーストーリーごとのタスク）
  - Tests for User Story 1
  - Implementation for User Story 1
  - Checkpoint: User Story 1完了時の確認項目
- Final Phase: Polish & Cross-Cutting Concerns
- Dependencies & Execution Order: 依存関係と実行順序
- Parallel Opportunities: 並列実行可能なタスク
- Implementation Strategy: MVP First、Incremental Delivery

**目的**: 実装タスクを細分化し、実行可能な単位に分割

**タスクの進め方**:
1. Phase 1（Setup）を完了
2. Phase 2（Foundational）を完了 ← **すべてのユーザーストーリーをブロック**
3. Phase 3以降（User Stories）を優先度順に実装
4. 各User Storyは独立してテスト可能
5. Final Phase（Polish）で品質向上

---

### checklists/（チェックリスト）

**Phase 0で生成**（`/speckit-specify`コマンドの一部、または手動作成）

**内容**:
- `requirements.md`: 機能要求のチェックリスト（spec.mdから自動生成）
- `testing.md`: テスト項目のチェックリスト
- `deployment.md`: デプロイ前のチェックリスト

**目的**: 実装漏れを防ぎ、品質を担保する

---

## GitHub Spec Kitのスラッシュコマンド

このプロジェクトでは、`.claude/commands/`に以下のスラッシュコマンドを配置:

- `/speckit-specify`: 機能仕様書（spec.md）を生成
- `/speckit-clarify`: 仕様の曖昧な点を明確化
- `/speckit-plan`: 実装計画（research.md, contracts/, quickstart.md, plan.md）を生成
- `/speckit-tasks`: タスクリスト（tasks.md）を生成
- `/speckit-implement`: 実装を開始（tasks.mdのタスクを順次実行）
- `/speckit-analyze`: 仕様書・計画書の整合性をチェック
- `/speckit-constitution`: プロジェクトの憲法（開発原則）を作成・更新
- `/speckit-checklist`: カスタムチェックリストを生成

**使い方**:
```bash
# Phase 0: 仕様書作成
/speckit-specify "一般ユーザー向けのメニュー一覧画面。期間限定メニューを一覧表示..."

# Phase 0: 曖昧な点を明確化
/speckit-clarify

# Phase 1: 実装計画作成
/speckit-plan

# Phase 2: タスクリスト生成
/speckit-tasks

# Phase 3: 実装開始
/speckit-implement
```

---

## このプロジェクトでの使用例

### 001-admin-panel（管理画面機能）

1. `/speckit-specify` → spec.md生成
2. `/speckit-clarify` → 仕様の曖昧な点を明確化
3. `/speckit-plan` → research.md, contracts/, quickstart.md, plan.md生成
4. `/speckit-tasks` → tasks.md生成
5. `/speckit-implement` → 実装（limimeshi-adminリポジトリ）

### 002-menu-list（メニュー一覧機能）

1. `/speckit-specify` → spec.md生成
2. `/speckit-plan` → research.md, contracts/, quickstart.md, plan.md生成
3. `/speckit-tasks` → tasks.md生成（37タスク）
4. `/speckit-implement` → 実装（limimeshi-webリポジトリ）

### 003-favorites（お気に入り登録機能）

1. `/speckit-specify` → spec.md生成
2. `/speckit-plan` → research.md, contracts/, quickstart.md, plan.md生成
3. `/speckit-tasks` → tasks.md生成（28タスク）
4. `/speckit-implement` → 実装（limimeshi-webリポジトリ）

---

## Constitution（開発原則）への準拠

すべての機能は、`memory/constitution.md`で定義された9つの開発原則に準拠する必要があります:

1. **Spec-Driven（仕様駆動開発）**: GitHub Spec Kit公式ワークフローに従う
2. **Test-First（テスト駆動）**: テストを先に書く
3. **Simplicity（シンプルさ優先）**: YAGNI原則、過度な抽象化を避ける
4. **Firebase-First**: Firebase優先のアーキテクチャ
5. **Legal Risk Zero（法的リスクゼロ）**: スクレイピング禁止、完全手動運用
6. **Mobile & Performance First**: モバイル対応、3秒以内のページ読み込み
7. **Cost Awareness（コスト意識）**: Firebase無料枠内での運用
8. **Security & Privacy First**: Firestore Security Rules、プライバシー保護
9. **Observability（可観測性）**: Firebase Console、Google Analytics 4

plan.mdの「Constitution Check」セクションで、各原則への準拠を確認しています。

---

## 関連ドキュメント

- [CLAUDE.md](../CLAUDE.md): AI向けプロジェクト情報
- [memory/constitution.md](../memory/constitution.md): プロジェクトの憲法（開発原則）
- [roadmap.md](../roadmap.md): プロジェクト全体のロードマップ
- [WRITING_STYLE_GUIDE.md](../WRITING_STYLE_GUIDE.md): ドキュメント記述ルール
- [templates/](../templates/): GitHub Spec Kitテンプレート
- [.claude/commands/](../.claude/commands/): GitHub Spec Kitスラッシュコマンド

---

**最終更新**: 2025-11-19
