# 新規リポジトリセットアップ

新しいlimimeshiリポジトリの初期セットアップを実行する

## 使用方法

**limimeshi-docsリポジトリから実行**：
```
/setup-new-repo [リポジトリ名]
```

例：
```
/setup-new-repo limimeshi-android
```

引数がない場合はリポジトリ名を質問する

## 前提条件

- 現在のワーキングディレクトリがlimimeshi-docs
- 対象リポジトリが兄弟ディレクトリに存在（`../[リポジトリ名]/`）

```
parent-directory/
├── limimeshi-docs/    ← ここから実行
└── target-repo/       ← セットアップ対象
```

## 作成するファイル

### シンボリックリンク（limimeshi-docs/shared/を参照）

| 作成先 | リンク先 |
|--------|---------|
| `.claude/settings.json` | `../../limimeshi-docs/shared/setup-new-repo/.claude/settings.json` |
| `.claude/commands/suggest-claude-md.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/commands/suggest-claude-md.md` |
| `.claude/commands/sync-shared-rules.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/commands/sync-shared-rules.md` |
| `.claude/skills/security-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/security-check.md` |
| `.claude/skills/style-guide-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/style-guide-check.md` |
| `docs/governance/docs-style-guide.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/docs-style-guide.md` |
| `docs/governance/shared-rules.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/shared-rules.md` |
| `.specify/.claude/commands/speckit-*.md` | `../../../../limimeshi-docs/shared/setup-new-repo/.specify/.claude/commands/speckit-*.md` |
| `.specify/templates/*.md` | `../../../limimeshi-docs/shared/setup-new-repo/.specify/templates/*.md` |

### 実体ファイル（template/setup-new-repo/からコピー）

| ファイル | 説明 |
|---------|------|
| `.specify/README.md` | Spec Kit使い方ガイド |
| `.specify/memory/constitution.md` | プロジェクトの憲法（カスタマイズ必要） |
| `.specify/specs/.gitkeep` | 機能仕様書ディレクトリ |
| `docs/README.md` | ドキュメントディレクトリ説明 |
| `docs/adr/README.md` | ADRディレクトリ |
| `docs/governance/README.md` | ガバナンスルール説明 |
| `docs/roadmap.md` | ロードマップ |
| `docs/CHANGELOG.md` | 変更履歴 |

## 実行手順

1. **引数確認**: リポジトリ名が指定されていなければ質問
2. **対象確認**: `../[リポジトリ名]/`の存在を確認
3. **作成内容表示**: 作成するファイル一覧を表示
4. **ユーザー確認**: 実行して良いか確認
5. **ディレクトリ作成**: 必要なディレクトリ構造を作成
6. **シンボリックリンク作成**: shared/への参照を作成
7. **ファイルコピー**: template/から実体ファイルをコピー
8. **置換処理**: `[REPOSITORY_NAME]`を実際のリポジトリ名に置換
9. **CLAUDE.md更新**: ディレクトリ構成セクションを追記
10. **結果報告**: 作成したファイル一覧を報告

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
- `commands/sync-shared-rules.md`
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

- 対象リポジトリが存在しない場合はエラー
- 既存ファイルがある場合は上書き前に確認
- constitution.mdは初期値としてコピー後、各リポジトリで独立管理
- セットアップ後はコミットを忘れずに
