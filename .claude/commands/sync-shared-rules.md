# 共通ルール同期

limimeshi-docsリポジトリから共通ルールを同期する。
docsリポジトリで更新があった際に、各実装リポジトリで実行する。

## 同期対象

以下のファイルを同期（constitution.md、Spec Kit、speckit-*コマンドは対象外）

### Claude Code設定

| ソース（limimeshi-docs） | コピー先 |
|-------------------------|---------|
| `.claude/settings.json` | `.claude/settings.json` |
| `.claude/skills/security-check.md` | `.claude/skills/security-check.md` |
| `.claude/skills/style-guide-check.md` | `.claude/skills/style-guide-check.md` |
| `.claude/commands/suggest-claude-md.md` | `.claude/commands/suggest-claude-md.md` |

### ガバナンスドキュメント

| ソース（limimeshi-docs） | コピー先 |
|-------------------------|---------|
| `governance/docs-style-guide.md` | `docs/governance/docs-style-guide.md` |
| `governance/shared-rules.md` | `docs/governance/shared-rules.md` |

## 同期対象外（初期コピーのみ）

以下は各リポジトリで独立管理するため同期しない：
- `constitution.md` - リポジトリ固有のカスタマイズ可
- `.specify/` - Spec Kit（各リポジトリの機能仕様）
- `speckit-*.md` - Spec Kitスラッシュコマンド（各リポジトリでカスタマイズ可）

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（通常は兄弟ディレクトリ）
2. 同期対象ファイルの差分を確認
3. 差分があるファイルを報告
4. ユーザーの確認後、ファイルを更新
5. 更新完了後、変更内容を報告

## 注意事項

- 既存ファイルとの差分を必ず確認してから上書き
- ローカルで独自の変更を加えている場合は要注意
- 同期後はコミットを忘れずに
