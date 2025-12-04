# 期間限定めし（リミメシ）ガバナンスリポジトリ

チェーン店の期間限定メニューに特化した情報プラットフォーム「期間限定めし（リミメシ）」の**ガバナンス専用リポジトリ**です。

> **Note**: 機能仕様書（specs）やSpec Kitファイルは各実装リポジトリに移行済みです。

## サービス概要

**正式名称**：期間限定めし
**略称**：リミメシ
**英語表記**：Limited Meshi（Limimeshi）

「なじみのチェーン店で新しい体験を楽しむ」をコンセプトに、日本の大手チェーン店で現在販売中の期間限定メニューを一覧できるサービス。

## このリポジトリの役割

| 役割 | 説明 |
|------|------|
| **ガバナンス** | プロジェクト全体の企画・方針・ルールを管理 |
| **共通ADR** | 複数リポジトリに影響する技術選定を記録 |
| **マスタードキュメント** | Constitution、docs-style-guideのマスターを保持 |

**実装・機能仕様は各実装リポジトリで管理**：
- [limimeshi-admin](https://github.com/shg25/limimeshi-admin)：管理画面（React Admin）
- [limimeshi-android](https://github.com/shg25/limimeshi-android)：Androidアプリ（Kotlin + Jetpack Compose）

## ドキュメント構成

| ディレクトリ/ファイル | 内容 |
|-----------|------|
| [adr/](./adr/) | 共通ADR（複数リポジトリに影響する技術選定） |
| [data-model/](./data-model/) | Firestoreスキーマ設計（→Phase3でlimimeshi-infraに移行予定） |
| [guides/](./guides/) | 本番環境セットアップガイド（→Phase3でlimimeshi-infraに移行予定） |
| [governance/](./governance/) | ガバナンスルール（constitution.md、docs-style-guide.md、shared-rules.md） |
| [planning/](./planning/) | Phase0企画ドキュメント（first-idea.md、lean-canvas.md、inception-deck.md） |
| [CLAUDE.md](./CLAUDE.md) | AI向けプロジェクト情報 |
| [roadmap.md](./roadmap.md) | プロジェクト全体のロードマップ |

## 新規リポジトリ作成・設定同期

Custom Slash Commandsを使用してセットアップ：

| コマンド | 用途 | 実行タイミング |
|---------|------|---------------|
| `/setup-new-repo` | 新規リポジトリの初期セットアップ | リポジトリ作成時に1回 |
| `/sync-shared-rules` | 共通ルールの同期 | docsリポジトリ更新後 |

**同期ポリシー**：
- **初期コピーのみ**：constitution.md、Spec Kit（各リポジトリで独立管理）
- **継続同期**：Claude Code設定、docs-style-guide.md、shared-rules.md

詳細は [.claude/commands/](./.claude/commands/) を参照

## 共通ADR一覧

| ADR | 内容 |
|-----|------|
| [ADR-001](./adr/001-use-firebase-for-backend.md) | Use Firebase for backend |
| [ADR-002](./adr/002-adopt-multi-repository-structure.md) | Adopt multi-repository structure |
| [ADR-003](./adr/003-use-react-admin-for-admin-panel.md) | Use React Admin for admin panel |
| [ADR-004](./adr/004-use-manual-data-entry-for-phase2.md) | Use manual data entry for Phase 2 |
| [ADR-005](./adr/005-deploy-using-firebase-hosting-multi-site.md) | Deploy using Firebase Hosting multi-site |

## 採用している手法・フレームワーク

| フェーズ | 手法・フレームワーク | 役割・目的 | 公式リンク |
|---------|-------------------|-----------|-----------|
| Phase0 | リーンキャンバス | ビジネスモデルの検証 | [Lean Canvas公式](https://leanstack.com/lean-canvas) |
| Phase0 | インセプションデッキ | プロジェクトの目的・優先順位の明確化 | [Agile Warrior解説](https://agilewarrior.wordpress.com/2010/11/06/the-inception-deck/) |
| Phase1 | GitHub Spec Kit | 仕様駆動開発（Spec-Driven Development） | [GitHub公式](https://github.com/github/spec-kit) |
| Phase1 | ADR | 技術選定の記録 | [ADR公式](https://adr.github.io/) |
| Phase1 | Firestore公式ベストプラクティス | データベース設計 | [Firebase公式](https://firebase.google.com/docs/firestore/best-practices) |
| 全体 | Keep a Changelog | 変更履歴の記録 | [Keep a Changelog公式](https://keepachangelog.com/ja/1.1.0/) |
| 全体 | Conventional Commits | コミットメッセージ規約 | [Conventional Commits公式](https://www.conventionalcommits.org/ja/v1.0.0/) |
| 全体 | Semantic Versioning | バージョン番号体系 | [Semantic Versioning公式](https://semver.org/lang/ja/) |

## リポジトリ構成

| リポジトリ | 役割 | 状態 |
|-------------|------|------|
| `limimeshi-docs` | ガバナンス（このリポジトリ） | ✅ 運用中 |
| `limimeshi-admin` | 管理画面（React Admin） | ✅ 実装完了 |
| `limimeshi-android` | Androidアプリ（Kotlin + Jetpack Compose） | 🚧 準備中 |
| `limimeshi-infra` | Firestore Rules/Indexes管理 | 📋 Phase3で作成予定 |
| `limimeshi-web` | Webアプリ | 📋 Phase3で作成予定 |

## コントリビューション

このプロジェクトは個人開発プロジェクトですが、フィードバックや提案は歓迎します。

詳細は [CONTRIBUTING.md](./CONTRIBUTING.md) をご覧ください。

## ライセンス

ドキュメントは [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) で公開しています。

## 作成者

重次弘規（[@shg25](https://github.com/shg25)）

---

**最終更新**：2025/12/03
