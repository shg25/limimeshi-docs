# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ja/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/lang/ja/).

## [Unreleased]

### Added
- 各リポジトリのREADME/CLAUDE.mdに前提条件・ディレクトリ構成セクションを追加
  - 前提条件：limimeshi-docsと同じ親ディレクトリに配置する必要がある旨を明記
  - ディレクトリ構成（ガバナンス関連）：シンボリックリンク/READMEコピー/リポジトリ固有ファイルを明確化
- template/setup-new-repo/配下に各種README.md追加（READMEコピー対象）
  - .specify/README.md：Spec Kit使い方ガイド
  - docs/README.md：ドキュメントディレクトリ説明
  - docs/adr/README.md：ADRディレクトリ説明
  - docs/governance/README.md：ガバナンスルール説明
- shared/setup-new-repo/ディレクトリ作成（マスターファイル管理用）
  - template/setup-new-repo/と同じ構造で直感的に
  - .claude/（Claude Code設定のマスター）
  - .specify/（Spec Kitファイルのマスター）
  - docs/governance/（ガバナンスドキュメントのマスター）
- template/setup-new-repo/ディレクトリ作成（新規リポジトリ一括セットアップ用）
  - .specify/（Spec Kit一式：memory、specs、templates、speckit-*コマンド）
  - .claude/（Claude Code設定：commands、skills、settings.json）
  - docs/（governance、adr、roadmap、CHANGELOG）
  - ※共有ファイルはshared/へのシンボリックリンク
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
- /sync-shared-rulesをdocsリポジトリから実行する形式に変更
  - 引数でリポジトリ名を指定：`/sync-shared-rules limimeshi-admin`
  - READMEコピー機能を追加（シンボリックリンク + READMEコピーの二重管理）
- 共有ファイル管理をシンボリックリンク方式に変更（コピー同期から自動同期へ）
  - 他リポジトリからlimimeshi-docs/shared/への相対シンボリックリンク
  - ファイル内容変更時の手動同期が不要に
- setup-new-repo.md：シンボリックリンク作成方式に変更、README/CLAUDE.md追記内容を追加
- sync-shared-rules.md：シンボリックリンク作成 + READMEコピーの両方に対応
- template/setup-new-repo/docs/adr/README.md：リポジトリ固有セクション（ADR一覧）を削除
- docs/governance/：docs-style-guide.mdとshared-rules.mdをshared/へのシンボリックリンクに変更
- .claude/：共有ファイル（settings.json、suggest-claude-md.md、skills/*）をshared/へのシンボリックリンクに変更
- docs/governance/constitution.md：削除（マスターはtemplate/setup-new-repo/.specify/memory/に移動）
- ディレクトリ構造を統一：governance/、adr/、roadmap.md、CHANGELOG.mdをdocs/以下に移動
- テンプレートをtemplate/配下に統合（docs/specify-template/ → template/setup-new-repo/）
- Claude Code用語を英語表記に統一（Custom Slash Commands / Agent Skills / Claude Code Hooks）
- memory/ → governance/ にリネーム
- データモデルをメニュー単位からキャンペーン単位に変更
- 開発優先順位：Webアプリ → Androidアプリ優先に変更
- specs/を各実装リポジトリに移行、limimeshi-docsをガバナンス専用に整理
- サービス名決定：「期間限定めし（リミメシ）」
- データ更新方針：スクレイピング廃止、完全手動運用に変更

### Fixed
- docs-style-guide.md：セクション番号15の重複を16に修正
- first-idea.md：廃止された「差分検知システム」への残存参照を削除
- CLAUDE.md：ADR-003の参照を修正（limimeshi-adminリポジトリに移行済み）
- roadmap.md：Phase2進捗率を0%→30%に更新（実態に合わせて修正）
