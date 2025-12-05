# 共有ファイル同期

既存リポジトリのシンボリックリンクとREADMEコピーを更新する

## 使用方法

**limimeshi-docsリポジトリから実行**：
```
/sync-shared-rules [リポジトリ名]
```

例：
```
/sync-shared-rules limimeshi-admin
```

引数がない場合はリポジトリ名を質問する

## 前提条件

- 現在のワーキングディレクトリがlimimeshi-docs
- 対象リポジトリが兄弟ディレクトリに存在（`../[リポジトリ名]/`）

```
parent-directory/
├── limimeshi-docs/    ← ここから実行
└── target-repo/       ← 同期対象
```

## シンボリックリンク対象

以下のファイルは `limimeshi-docs/shared/setup-new-repo/` へのシンボリックリンク。
パスはファイルの深さにより異なる。

### Claude Code設定

| シンボリックリンク | リンク先 |
|------------------|---------|
| `.claude/settings.json` | `../../limimeshi-docs/shared/setup-new-repo/.claude/settings.json` |
| `.claude/skills/security-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/security-check.md` |
| `.claude/skills/style-guide-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/style-guide-check.md` |
| `.claude/commands/suggest-claude-md.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/commands/suggest-claude-md.md` |

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

以下のファイルはGitHubでの表示用にコピー。limimeshi-docsで管理。

| コピー先 | コピー元 |
|---------|---------|
| `.specify/README.md` | `template/setup-new-repo/.specify/README.md` |
| `docs/README.md` | `template/setup-new-repo/docs/README.md` |
| `docs/adr/README.md` | `template/setup-new-repo/docs/adr/README.md` |
| `docs/governance/README.md` | `template/setup-new-repo/docs/governance/README.md` |

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

1. **引数確認**: リポジトリ名が指定されていなければ質問
2. **対象確認**: `../[リポジトリ名]/`の存在を確認
3. **シンボリックリンク確認**: 壊れたリンクを検出
4. **シンボリックリンク作成**: 不足しているリンクを作成
5. **READMEコピー**: template/から対象リポジトリへコピー
6. **結果報告**: 変更したファイル一覧を報告

## 注意事項

- 対象リポジトリが存在しない場合はエラー
- 既存の実体ファイルは確認してからシンボリックリンクに置換
- 同期後は対象リポジトリでコミットを忘れずに
