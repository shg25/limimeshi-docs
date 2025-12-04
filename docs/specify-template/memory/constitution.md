# [PROJECT_NAME] Constitution

<!--
  このファイルはプロジェクトの「憲法」テンプレートです。

  ## マスターファイル
  limimeshi-docsリポジトリのマスター: docs/governance/constitution.md

  ## 使い方
  1. 新規リポジトリ作成時にこのファイルを .specify/memory/ にコピー
  2. [PROJECT_NAME] を実際のプロジェクト名に置換
  3. 開発原則をプロジェクトに合わせてカスタマイズ
  4. 各リポジトリで独立管理（マスターとの同期は不要）

  ## 注意
  - 憲法の修正にはドキュメント化・承認・移行計画が必要
  - 重大な修正はADRで記録
-->

## 開発原則（Core Principles）

開発原則は3つのレイヤーで構成
- 基礎原則（Fundamental）
- プロジェクト固有の制約（Project Constraints）
- 品質基準（Quality Standards）

### 基礎原則（Fundamental）

#### I. Spec-Driven（仕様駆動開発）

**出典**：[GitHub Spec Kit公式](https://github.com/github/spec-kit)

仕様駆動開発を実践
- GitHub Spec Kit公式ワークフロー（`/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks`）に従う
- 仕様が王様、コードは従者
- 全ての機能は仕様書（spec.md、plan.md、tasks.md）から開始

#### II. Test-First（テスト駆動）※非交渉原則

**出典**：GitHub Spec Kit公式例

テスト駆動開発（TDD）は絶対原則
- テストを先に書く → ユーザー承認 → テストが失敗することを確認 → 実装
- Red-Green-Refactorサイクルを厳守
- 全ての機能追加・変更はテストから開始

#### III. Simplicity（シンプルさ優先）

**出典**：GitHub Spec Kit公式例

シンプルな設計を最優先
- YAGNI原則：必要になるまで実装しない
- 過度な抽象化を避ける（Anti-Abstraction）
- フレームワークの機能を直接使用し、不要なラッパーを作らない
- 複雑性の追加には正当な理由が必要

### プロジェクト固有の制約（Project Constraints）

<!-- 以下はプロジェクトに応じてカスタマイズ -->

#### IV. [CONSTRAINT_1_NAME]

**出典**：[参照元]

[制約の説明]

#### V. [CONSTRAINT_2_NAME]

**出典**：[参照元]

[制約の説明]

### 品質基準（Quality Standards）

<!-- 以下はプロジェクトに応じてカスタマイズ -->

#### VI. [QUALITY_1_NAME]

**出典**：[参照元]

[品質基準の説明]

#### VII. [QUALITY_2_NAME]

**出典**：[参照元]

[品質基準の説明]

## 技術選定方針

<!-- プロジェクトの技術スタックを記載 -->

### フロントエンド

- [技術選定]

### バックエンド

- [技術選定]

### デプロイ・CI/CD

- [技術選定]

## ドキュメント管理

### 記述ルール

- docs-style-guide.mdに従う
- 「。」は注意書き・免責事項のみ
- 箇条書き・説明文は「〜する」で統一
- 太字は必要最小限（サービス名、技術用語の初出、重要なルール）

### ADR（Architecture Decision Records）

- 技術選定・設計決定は必ずADRに記録
- Context（背景）→ Decision（決定）→ Consequences（結果）の形式
- docs/adr/ディレクトリに番号付きで管理

## ガバナンス

### 憲法の優先順位

- この憲法は全ての開発プラクティスに優先する
- 憲法の修正には、ドキュメント化・承認・移行計画が必要

### レビュー・承認

- 全てのPR・レビューは憲法への準拠を確認
- 複雑性の追加には正当な理由が必要
- 開発時の実行ガイダンスは CLAUDE.md を参照

### 憲法の更新

- 軽微な修正：typo修正、文言の明確化
- 重大な修正：原則の追加・削除、技術選定の変更（ADR必須）

---

**Version**: 1.0.0 | **Ratified**: [DATE] | **Last Amended**: [DATE]
