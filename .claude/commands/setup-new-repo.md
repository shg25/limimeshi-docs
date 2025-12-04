# 新規リポジトリセットアップ

新しいlimimeshiリポジトリを作成する際の初期セットアップを実行する

## 対象リポジトリ

現在のワーキングディレクトリが対象リポジトリ
limimeshi-docsリポジトリの`template/setup-new-repo/`から必要なファイルをコピーする

## テンプレート内容

**ソース**: `limimeshi-docs/template/setup-new-repo/`

```
template/setup-new-repo/
├── .specify/              # GitHub Spec Kit（仕様駆動開発）
│   ├── memory/
│   │   └── constitution.md  # プロジェクトの憲法
│   ├── specs/               # 機能仕様書（空）
│   ├── templates/           # 仕様書テンプレート
│   └── .claude/
│       └── commands/        # speckit-*.md コマンド
├── .claude/               # Claude Code設定
│   ├── commands/
│   │   └── suggest-claude-md.md
│   ├── skills/
│   └── settings.json
├── docs/                  # ガバナンスドキュメント
│   ├── governance/
│   ├── adr/
│   ├── roadmap.md
│   └── CHANGELOG.md
└── README.md              # テンプレート説明（コピー後に削除）
```

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（通常は兄弟ディレクトリ）
2. `template/setup-new-repo/`の内容をまるごとコピー
3. `template/setup-new-repo/README.md`を削除（テンプレート説明用なので不要）
4. `.specify/memory/constitution.md`をプロジェクトに合わせてカスタマイズ
5. `docs/roadmap.md`の`[REPOSITORY_NAME]`を実際のリポジトリ名に変更
6. CLAUDE.mdに「ディレクトリ構成（ガバナンス関連）」セクションを追記
7. コピー完了後、差分を報告

## CLAUDE.mdに追記する内容

```markdown
## ディレクトリ構成（ガバナンス関連）

本リポジトリは limimeshi-docs のガバナンスルールに従う

### docs/governance/
limimeshi-docsから同期されるガバナンスドキュメント：
- `docs-style-guide.md`：ドキュメント記述ルール
- `shared-rules.md`：複数リポジトリ共通ルール

### .claude/
Claude Code設定：
- `commands/`：Custom Slash Commands
- `skills/`：Agent Skills
- `settings.json`：Claude Code Hooks設定

### .specify/
GitHub Spec Kit（仕様駆動開発）：
- `memory/constitution.md`：プロジェクトの憲法
- `specs/`：機能仕様書
- `templates/`：仕様書テンプレート

### 同期について
- `docs/governance/` と `.claude/`（speckit-*以外）は limimeshi-docs から同期
- 同期は `/sync-shared-rules` コマンドで実行
- 詳細は limimeshi-docs/README.md を参照
```

## 注意事項

- constitution.mdは初期値としてコピー。その後は各リポジトリで独立管理
- speckit-*コマンドは各リポジトリでカスタマイズ可能
- その他のClaude Code設定とガバナンスドキュメントは `/sync-shared-rules` で継続同期
- 既存ファイルがある場合は上書き前に確認
