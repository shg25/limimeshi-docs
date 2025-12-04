# 新規リポジトリセットアップ

新しいlimimeshiリポジトリを作成する際の初期セットアップを実行する

## 前提条件

limimeshi-docsリポジトリが兄弟ディレクトリにclone済みであること：
```
parent-directory/
├── limimeshi-docs/    ← 必須
└── new-repo/          ← セットアップ対象
```

## 作成するファイル

### シンボリックリンク（limimeshi-docs/shared/setup-new-repo/を参照）

以下は `../limimeshi-docs/shared/setup-new-repo/` へのシンボリックリンクとして作成：

```
.claude/
├── commands/suggest-claude-md.md → ../limimeshi-docs/shared/...
├── skills/security-check.md → ../limimeshi-docs/shared/...
├── skills/style-guide-check.md → ../limimeshi-docs/shared/...
└── settings.json → ../limimeshi-docs/shared/...

.specify/
├── .claude/commands/speckit-*.md → ../limimeshi-docs/shared/...
└── templates/*.md → ../limimeshi-docs/shared/...

docs/governance/
├── docs-style-guide.md → ../limimeshi-docs/shared/...
└── shared-rules.md → ../limimeshi-docs/shared/...
```

### 実体ファイル（各リポジトリ固有）

以下は `template/setup-new-repo/` から実体としてコピー：

```
.specify/
├── memory/constitution.md   # カスタマイズ必要
└── specs/.gitkeep           # リポジトリ固有の仕様書

docs/
├── adr/README.md            # リポジトリ固有のADR
├── roadmap.md               # リポジトリ固有のロードマップ
└── CHANGELOG.md             # リポジトリ固有の変更履歴
```

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（兄弟ディレクトリ `../limimeshi-docs/`）
2. 必要なディレクトリ構造を作成
3. シンボリックリンクを作成（shared/へ参照）
4. 実体ファイルをコピー（template/から）
5. `.specify/memory/constitution.md`をプロジェクトに合わせてカスタマイズ
6. `docs/roadmap.md`の`[REPOSITORY_NAME]`を実際のリポジトリ名に変更
7. CLAUDE.mdに「ディレクトリ構成（ガバナンス関連）」セクションを追記
8. セットアップ完了後、差分を報告

## CLAUDE.mdに追記する内容

```markdown
## 前提条件

このリポジトリは limimeshi-docs と同じ親ディレクトリに配置する必要がある：
\`\`\`
parent-directory/
├── limimeshi-docs/    ← 必須
└── this-repo/
\`\`\`

## ディレクトリ構成（ガバナンス関連）

本リポジトリは limimeshi-docs のガバナンスルールに従う

### docs/governance/
limimeshi-docsへのシンボリックリンク（自動同期）：
- `docs-style-guide.md`：ドキュメント記述ルール
- `shared-rules.md`：複数リポジトリ共通ルール

### .claude/
Claude Code設定（limimeshi-docsへのシンボリックリンク）：
- `commands/suggest-claude-md.md`
- `skills/`：Agent Skills
- `settings.json`：Claude Code Hooks設定

### .specify/
GitHub Spec Kit（仕様駆動開発）：
- `memory/constitution.md`：プロジェクトの憲法（実体、カスタマイズ可）
- `specs/`：機能仕様書（実体、リポジトリ固有）
- `.claude/commands/`：speckit-*コマンド（シンボリックリンク）
- `templates/`：仕様書テンプレート（シンボリックリンク）

### 同期について
- シンボリックリンクのファイルは limimeshi-docs を更新すれば自動反映
- constitution.mdとspecs/は各リポジトリで独立管理
```

## 注意事項

- limimeshi-docsが兄弟ディレクトリにない場合、シンボリックリンクが壊れる
- constitution.mdは初期値としてコピー。その後は各リポジトリで独立管理
- specs/内の機能仕様書は各リポジトリ固有
- 既存ファイルがある場合は上書き前に確認
