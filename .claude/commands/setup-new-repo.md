# 新規リポジトリセットアップ

新しいlimimeshiリポジトリを作成する際の初期セットアップを実行する。

## 対象リポジトリ

現在のワーキングディレクトリが対象リポジトリ。
limimeshi-docsリポジトリから必要なファイルをコピーする。

## コピー対象

### 1. Spec Kit構造（初期値、その後は各リポジトリで独立管理）

```
.specify/
├── memory/
│   └── constitution.md # ← limimeshi-docs/docs/governance/constitution.md
├── specs/              # 機能仕様書（空）
└── templates/          # テンプレート
```

### 2. Claude Code設定

```
.claude/
├── commands/
│   ├── speckit-specify.md     # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-plan.md        # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-tasks.md       # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-implement.md   # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-clarify.md     # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-analyze.md     # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-checklist.md   # 初期コピーのみ（カスタマイズ可）
│   ├── speckit-constitution.md # 初期コピーのみ（カスタマイズ可）
│   └── suggest-claude-md.md   # 継続同期対象
├── settings.json              # 継続同期対象
└── skills/
    ├── security-check.md      # 継続同期対象
    └── style-guide-check.md   # 継続同期対象
```

**同期ポリシー**：
- `speckit-*` コマンド：初期コピーのみ（各リポジトリでカスタマイズ可）
- その他：継続同期対象（`/sync-shared-rules`で同期）

### 3. ガバナンスドキュメント（継続同期対象）

```
docs/
└── governance/
    ├── docs-style-guide.md  # ← limimeshi-docs/docs/governance/docs-style-guide.md
    └── shared-rules.md      # ← limimeshi-docs/docs/governance/shared-rules.md
```

### 4. プロジェクト管理ファイル（新規作成）

```
docs/
├── roadmap.md    # リポジトリ固有のロードマップ
└── CHANGELOG.md  # 変更履歴（Keep a Changelog形式）
```

**roadmap.md**：リポジトリ固有のタスク・進捗を管理

**CHANGELOG.md**：Keep a Changelog形式で作成
```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ja/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/lang/ja/).

## [Unreleased]

### Added
- 初期セットアップ
```

### 5. ADRディレクトリ（新規作成）

```
docs/
└── adr/
    └── README.md  # ADR説明とテンプレート
```

**README.md**：以下のテンプレートで作成
```markdown
# Architecture Decision Records（ADR）

このリポジトリ固有の技術選定を記録

## ADRとは

Architecture Decision Records（ADR）は、アーキテクチャに関する重要な決定とその理由を記録するドキュメント

**出典**: Michael Nygard "[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)" (2011)

## ADR一覧

（ADRを作成したらここに追加）

## 命名規則

`NNN-short-title.md`形式で命名
- **NNN**: 連番（001, 002, 003...）
- **short-title**: 短いタイトル（kebab-case）

**タイトルパターン**:
- `Use [technology] for [purpose]`
- `Adopt [approach]`
- `Choose [option]`

## 共通ADR

複数リポジトリに影響するADRは [limimeshi-docs/docs/adr/](https://github.com/shg25/limimeshi-docs/tree/main/docs/adr) を参照
```

### 6. CLAUDE.md更新

対象リポジトリのCLAUDE.mdに以下の内容を追記：

```markdown
## ディレクトリ構成（ガバナンス関連）

本リポジトリは limimeshi-docs のガバナンスルールに従う。

### docs/governance/
limimeshi-docsから同期されるガバナンスドキュメント：
- `docs-style-guide.md`：ドキュメント記述ルール
- `shared-rules.md`：複数リポジトリ共通ルール

### .claude/
Claude Code設定：
- `commands/`：Custom Slash Commands
- `skills/`：Agent Skills
- `settings.json`：Claude Code Hooks設定

### 同期について
- `docs/governance/` と `.claude/`（speckit-*以外）は limimeshi-docs から同期
- 同期は `/sync-shared-rules` コマンドで実行
- 詳細は limimeshi-docs/README.md を参照
```

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（通常は兄弟ディレクトリ）
2. `docs/governance/`、`docs/adr/` ディレクトリを作成
3. `.claude/commands/`、`.claude/skills/` ディレクトリを作成
4. 上記ファイルを順番にコピー
5. constitution.mdの内容を確認し、リポジトリ固有のカスタマイズが必要か確認
6. `docs/roadmap.md` を作成（リポジトリ固有の内容で）
7. `docs/CHANGELOG.md` を作成（Keep a Changelog形式）
8. `docs/adr/README.md` を作成（テンプレートで）
9. CLAUDE.mdに「ディレクトリ構成（ガバナンス関連）」セクションを追記
10. コピー完了後、差分を報告

## 注意事項

- constitution.mdは初期値としてコピー。その後は各リポジトリで独立管理
- speckit-*コマンドは初期コピーのみ。各リポジトリでカスタマイズ可能
- その他のClaude Code設定とガバナンスドキュメントは `/sync-shared-rules` で継続同期
- 既存ファイルがある場合は上書き前に確認
