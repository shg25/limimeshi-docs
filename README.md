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
| [planning/](./planning/) | Phase0企画ドキュメント（first-idea.md、lean-canvas.md、inception-deck.md） |
| [specs/](./specs/) | 機能設計書（GitHub Spec Kit形式） |
| [adr/](./adr/) | Architecture Decision Records（技術選定の記録） |
| [api/](./api/) | API仕様書 |
| [data-model/](./data-model/) | データモデル詳細設計 |
| [roadmap.md](./roadmap.md) | プロジェクト全体のロードマップ、フェーズ管理、進捗状況 |
| [WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md) | ドキュメント記述ルール |

## 開発方針

- ドキュメント駆動開発：実装前に企画・設計を詳細に文書化
- AI活用前提：AIエージェントが読み取りやすいMarkdown形式で作成
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

**最終更新**：2025/10/26
