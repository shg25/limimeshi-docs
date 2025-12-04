# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ja/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/lang/ja/).

## [Unreleased]

### Added
- Keep a Changelog / Conventional Commits / Semantic Versioning 採用を宣言
- Custom Slash Commands作成（`/setup-new-repo`、`/sync-shared-rules`、`/suggest-claude-md`）
- Agent Skills作成（`security-check.md`、`style-guide-check.md`）
- Claude Code Hooks設定（`.claude/settings.json`）
- governance/shared-rules.md作成（複数リポジトリ共通ルール）
- GitHub Spec Kit形式の機能設計（001-admin-panel、002-chain-list、003-favorites）
- ADR-001〜005作成（Firebase、マルチリポジトリ、React Admin、手動運用、Firebase Hosting）
- Firestoreデータベース設計（firestore-collections.md、security-rules.md、indexes.md）
- Constitution（governance/constitution.md）
- first-idea.md、lean-canvas.md、inception-deck.md（企画ドキュメント）
- CLAUDE.md（AI向けプロジェクト情報）
- governance/docs-style-guide.md（ドキュメント記述ルール）

### Changed
- Claude Code用語を英語表記に統一（Custom Slash Commands / Agent Skills / Claude Code Hooks）
- memory/ → governance/ にリネーム
- データモデルをメニュー単位からキャンペーン単位に変更
- 開発優先順位：Webアプリ → Androidアプリ優先に変更
- specs/を各実装リポジトリに移行、limimeshi-docsをガバナンス専用に整理
- サービス名決定：「期間限定めし（リミメシ）」
- データ更新方針：スクレイピング廃止、完全手動運用に変更
