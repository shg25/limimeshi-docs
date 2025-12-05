# 共有ファイル同期（シンボリックリンク）

既存リポジトリのシンボリックリンクを更新する。
shared/にファイルが追加・削除された場合、または既存リポジトリをシンボリックリンク方式に移行する際に使用する。

## 前提条件

limimeshi-docsリポジトリが兄弟ディレクトリにあること：
```
parent-directory/
├── limimeshi-docs/    ← ソース
└── target-repo/       ← 同期対象（現在のワーキングディレクトリ）
```

## シンボリックリンク対象

以下のファイルは `limimeshi-docs/shared/setup-new-repo/` へのシンボリックリンク。
パスはファイルの深さにより異なる（リポジトリルートからlimimeshi-docsへは`../limimeshi-docs/`）。

### Claude Code設定

| シンボリックリンク | リンク先 |
|------------------|---------|
| `.claude/settings.json` | `../../limimeshi-docs/shared/setup-new-repo/.claude/settings.json` |
| `.claude/skills/security-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/security-check.md` |
| `.claude/skills/style-guide-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/style-guide-check.md` |
| `.claude/commands/suggest-claude-md.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/commands/suggest-claude-md.md` |
| `.claude/commands/sync-shared-rules.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/commands/sync-shared-rules.md` |

### ガバナンスドキュメント

| シンボリックリンク | リンク先 |
|------------------|---------|
| `docs/governance/docs-style-guide.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/docs-style-guide.md` |
| `docs/governance/shared-rules.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/shared-rules.md` |

### Spec Kitファイル

| シンボリックリンク | リンク先 |
|------------------|---------|
| `.specify/.claude/commands/speckit-*.md` | `../../../../limimeshi-docs/shared/setup-new-repo/.specify/.claude/commands/speckit-*.md` |
| `.specify/templates/*.md` | `../../../limimeshi-docs/shared/setup-new-repo/.specify/templates/*.md` |

## READMEコピー対象（編集不可）

以下のファイルはGitHubでの表示用にコピー。limimeshi-docsで管理し、このコマンドで同期。

| コピー先 | コピー元 |
|---------|---------|
| `.specify/README.md` | `../limimeshi-docs/template/setup-new-repo/.specify/README.md` |
| `docs/README.md` | `../limimeshi-docs/template/setup-new-repo/docs/README.md` |
| `docs/adr/README.md` | `../limimeshi-docs/template/setup-new-repo/docs/adr/README.md` |
| `docs/governance/README.md` | `../limimeshi-docs/template/setup-new-repo/docs/governance/README.md` |

## 同期対象外（各リポジトリ固有）

以下は実体ファイルとして各リポジトリで独立管理：
- `constitution.md` - リポジトリ固有のカスタマイズ
- `specs/` - 機能仕様書（リポジトリ固有）
- `roadmap.md` - リポジトリ固有のロードマップ
- `CHANGELOG.md` - リポジトリ固有の変更履歴

## 実行タイミング

シンボリックリンク方式では、**ファイル内容の変更は自動反映**されるため、以下の場合のみ実行：

1. **シンボリックリンク追加・削除時**: shared/にファイルが追加・削除された場合
2. **READMEファイル更新時**: template/setup-new-repo/内のREADMEが更新された場合
3. **移行時**: コピー方式からシンボリックリンク方式への移行

## 実行手順

1. limimeshi-docsリポジトリの場所を確認（`../limimeshi-docs/`）
2. 現在のシンボリックリンク状態を確認
3. 壊れたシンボリックリンクを検出
4. 不足しているシンボリックリンクを作成
5. READMEファイルをコピー（template/から）
6. 変更完了後、差分を報告

## 注意事項

- limimeshi-docsが兄弟ディレクトリにない場合は実行不可
- 既存の実体ファイルは確認してからシンボリックリンクに置換
- 同期後はコミットを忘れずに
