# 新規リポジトリセットアップ

新しいlimimeshiリポジトリを作成する際の初期セットアップを実行する。

## 対象リポジトリ

現在のワーキングディレクトリが対象リポジトリ。
limimeshi-docsリポジトリから必要なファイルをコピーする。

## コピー対象

### 1. Spec Kit構造（初期値、その後は各リポジトリで独立管理）

```
.specify/
├── .claude/commands/   # スラッシュコマンド
├── memory/
│   └── constitution.md # ← limimeshi-docs/governance/constitution.md
├── specs/              # 機能仕様書（空）
└── templates/          # テンプレート
```

### 2. Claude Code設定（継続同期対象）

```
.claude/
├── settings.json       # ← limimeshi-docs/.claude/settings.json
└── skills/
    ├── security-check.md      # ← limimeshi-docs/.claude/skills/security-check.md
    └── style-guide-check.md   # ← limimeshi-docs/.claude/skills/style-guide-check.md
```

### 3. ガバナンスドキュメント参照用コピー（継続同期対象）

```
docs/
└── governance/
    ├── docs-style-guide.md  # ← limimeshi-docs/governance/docs-style-guide.md
    └── shared-rules.md      # ← limimeshi-docs/governance/shared-rules.md
```

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（通常は兄弟ディレクトリ）
2. 上記ファイルを順番にコピー
3. constitution.mdの内容を確認し、リポジトリ固有のカスタマイズが必要か確認
4. コピー完了後、差分を報告

## 注意事項

- constitution.mdは初期値としてコピー。その後は各リポジトリで独立管理
- Claude Code設定とガバナンスドキュメントは `/sync-shared-rules` で継続同期可能
- 既存ファイルがある場合は上書き前に確認
