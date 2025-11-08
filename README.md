# 期間限定めし（リミメシ）企画・設計ドキュメント

チェーン店の期間限定メニューに特化した情報プラットフォーム「期間限定めし（リミメシ）」の企画・設計ドキュメントリポジトリです。

## サービス概要

**正式名称**：期間限定めし
**略称**：リミメシ
**英語表記**：Limited Meshi（Limimeshi）

「なじみのチェーン店で新しい体験を楽しむ」をコンセプトに、日本の大手チェーン店で現在販売中の期間限定メニューを一覧できるサービスです。

## ドキュメント構成

| ディレクトリ/ファイル | 内容 |
|-----------|------|
| [.claude/](./.claude/) | Claude Code設定（スラッシュコマンド） |
| [adr/](./adr/) | Architecture Decision Records（技術選定の記録） |
| [api/](./api/) | API仕様書 |
| [data-model/](./data-model/) | データモデル詳細設計 |
| [memory/](./memory/) | Constitution（憲法）：開発原則、技術選定方針、品質基準 |
| [planning/](./planning/) | Phase0企画ドキュメント（first-idea.md、lean-canvas.md、inception-deck.md） |
| [scripts/](./scripts/) | GitHub Spec Kitスクリプト（bash/PowerShell） |
| [specs/](./specs/) | 機能設計書（GitHub Spec Kit形式） |
| [specs-old/](./specs-old/) | 旧仕様書のバックアップ |
| [templates/](./templates/) | GitHub Spec Kitテンプレート（spec、plan、tasks） |
| [CLAUDE.md](./CLAUDE.md) | AI向けプロジェクト情報 |
| [MIGRATION_TO_SPEC_KIT.md](./MIGRATION_TO_SPEC_KIT.md) | GitHub Spec Kit適合計画 |
| [README.md](./README.md) | プロジェクト概要（このファイル） |
| [roadmap.md](./roadmap.md) | プロジェクトロードマップ |
| [WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md) | ドキュメント記述ルール |

## 採用している手法・フレームワーク

このプロジェクトは、業界標準または公式推奨の手法のみを使用しています。

| フェーズ | 手法・フレームワーク | 役割・目的 | 公式リンク |
|---------|-------------------|-----------|-----------|
| Phase0 | リーンキャンバス（Lean Canvas） | ビジネスモデルの検証 | [Lean Canvas公式](https://leanstack.com/lean-canvas) |
| Phase0 | インセプションデッキ（Inception Deck） | プロジェクトの目的・優先順位の明確化 | [Agile Warrior解説](https://agilewarrior.wordpress.com/2010/11/06/the-inception-deck/) |
| Phase1 | GitHub Spec Kit | 仕様駆動開発（Spec-Driven Development） | [GitHub公式](https://github.com/github/spec-kit) |
| Phase1 | ADR（Architecture Decision Records） | 技術選定の記録 | [ADR公式](https://adr.github.io/) |
| Phase1 | Firestore公式ベストプラクティス | データベース設計 | [Firebase公式](https://firebase.google.com/docs/firestore/best-practices) |

**詳細**：各手法の詳細な適用方法は [roadmap.md](./roadmap.md) を参照

## GitHub Spec Kit（仕様駆動開発）

このプロジェクトは **GitHub Spec Kit** を採用しています（2025/10/27）。

GitHub Spec Kitは、GitHubが公式に提供する仕様駆動開発（Spec-Driven Development）のためのツールキット/テンプレート集です。

### 主要な構成要素

- **Constitution（憲法）**: [memory/constitution.md](./memory/constitution.md)
  - プロジェクトの開発原則、技術選定方針、品質基準を定義
  - 全ての実装・設計はこの憲法に準拠

- **スラッシュコマンド**: [.claude/commands/](./.claude/commands/)
  - `/speckit-specify <機能の説明>`: 機能仕様書（spec.md）を生成
  - `/speckit-plan`: 実装計画（plan.md）を生成
  - `/speckit-tasks`: タスクリスト（tasks.md）を生成
  - `/speckit-clarify`: 仕様の曖昧な点を明確化
  - `/speckit-implement`: 実装開始

- **テンプレート**: [templates/](./templates/)
  - spec-template.md：機能仕様書テンプレート
  - plan-template.md：実装計画テンプレート
  - tasks-template.md：タスクリストテンプレート

- **スクリプト**: [scripts/](./scripts/)
  - スラッシュコマンドから呼び出されるbash/PowerShellスクリプト
  - 機能番号の自動採番、ディレクトリ作成などを自動化

### 参考リンク

- [GitHub Spec Kit公式リポジトリ](https://github.com/github/spec-kit)
- [適合計画の詳細](./MIGRATION_TO_SPEC_KIT.md)

## 開発方針

- 仕様駆動開発（Spec-Driven Development）：**GitHub Spec Kit** 採用（2025/10/27）
- ドキュメント駆動開発：実装前に企画・設計を詳細に文書化
- AI活用前提：Claude Code + スラッシュコマンドで効率化
- Constitution（憲法）：開発原則を [memory/constitution.md](./memory/constitution.md) で定義
- 公開リポジトリ：透明性を重視し、企画段階から公開

## 技術スタック

- バックエンド：Firebase (Auth, Firestore, Functions, Hosting, Scheduler)
- フロントエンド：React
- 管理画面：React Admin
- モバイル（将来）：Expo + React Native / SwiftUI / Jetpack Compose

## リポジトリ構成（予定）

| リポジトリ | 役割 | 備考 |
|-------------|------|------|
| `limimeshi-web` | 一般向けWebアプリ | Hosting:web |
| `limimeshi-admin` | 管理UI | Hosting:admin |
| `limimeshi-jobs` | 定期処理（季節メニューリマインダー等） | Functions（onSchedule） |
| `limimeshi-docs` | 企画・設計・ADR | 公開リポジトリ（このリポジトリ） |

## コントリビューション

このプロジェクトは個人開発プロジェクトですが、フィードバックや提案は歓迎します。

詳細は [CONTRIBUTING.md](./CONTRIBUTING.md) をご覧ください。

## ライセンス

ドキュメントは [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) で公開しています。

## 作成者

重次弘規（[@shg25](https://github.com/shg25)）

---

**最終更新**：2025/10/27
