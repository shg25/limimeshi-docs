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
| `.claude/skills/security-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/security-check.md` |
| `.claude/skills/style-guide-check.md` | `../../../limimeshi-docs/shared/setup-new-repo/.claude/skills/style-guide-check.md` |
| `docs/governance/docs-style-guide.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/docs-style-guide.md` |
| `docs/governance/shared-rules.md` | `../../../limimeshi-docs/shared/setup-new-repo/docs/governance/shared-rules.md` |
| `.specify/.claude/commands/speckit-*.md` | `../../../../limimeshi-docs/shared/setup-new-repo/.specify/.claude/commands/speckit-*.md` |
| `.specify/templates/*.md` | `../../../limimeshi-docs/shared/setup-new-repo/.specify/templates/*.md` |

### READMEコピー（編集不可、limimeshi-docsで管理）

GitHubでの表示用にコピー。編集はlimimeshi-docsで行い、`/sync-shared-rules`で同期。

| ファイル | 説明 |
|---------|------|
| `.specify/README.md` | Spec Kit使い方ガイド |
| `docs/README.md` | ドキュメントディレクトリ説明 |
| `docs/adr/README.md` | ADRディレクトリ説明 |
| `docs/governance/README.md` | ガバナンスルール説明 |

### リポジトリ固有ファイル（編集可）

各リポジトリで独立管理。自由に編集可能。

| ファイル | 説明 |
|---------|------|
| `.specify/memory/constitution.md` | プロジェクトの憲法（カスタマイズ必要） |
| `.specify/specs/.gitkeep` | 機能仕様書ディレクトリ |
| `docs/roadmap.md` | ロードマップ |
| `docs/CHANGELOG.md` | 変更履歴 |

## git-flow初期化

実装リポジトリ（docs以外）では**git-flow**を採用。セットアップ時に自動で初期化する。

### 実行内容

```bash
# git-flow初期化（デフォルト設定）
git flow init -d

# developブランチをリモートにpush
git push -u origin develop
```

### ブランチ構成

| ブランチ | 用途 | マージ先 |
|---------|------|---------|
| `main` | 本番環境（リリース済み） | - |
| `develop` | 開発環境（次期リリース準備） | main |
| `feature/*` | 機能開発 | develop |
| `release/*` | リリース準備 | main + develop |
| `hotfix/*` | 緊急修正 | main + develop |

### 参考

詳細は [CONTRIBUTING.md](https://github.com/shg25/limimeshi-docs/blob/main/CONTRIBUTING.md) を参照

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
10. **git-flow初期化**: `git flow init -d`を実行
11. **developブランチpush**: `git push -u origin develop`
12. **結果報告**: 作成したファイル一覧を報告

## CLAUDE.mdに追記する内容

```markdown
## 前提条件

このリポジトリは limimeshi-docs と同じ親ディレクトリに配置する必要がある：
\`\`\`
parent-directory/
├── limimeshi-docs/    ← 必須
└── this-repo/
\`\`\`

## ブランチ戦略

このリポジトリは**git-flow**を採用

### ブランチ構成
| ブランチ | 用途 |
|---------|------|
| `main` | 本番環境 |
| `develop` | 開発環境 |
| `feature/*` | 機能開発 |
| `release/*` | リリース準備 |
| `hotfix/*` | 緊急修正 |

### 基本フロー
1. `develop`から`feature/xxx`ブランチを作成
2. 作業完了後、`develop`へPR
3. リリース時は`release/x.x`→`main`にマージ

詳細は [CONTRIBUTING.md](https://github.com/shg25/limimeshi-docs/blob/main/CONTRIBUTING.md) を参照

## ディレクトリ構成（ガバナンス関連）

本リポジトリは limimeshi-docs のガバナンスルールに従う

### docs/
- `roadmap.md`：ロードマップ（実体、リポジトリ固有）
- `CHANGELOG.md`：変更履歴（実体、リポジトリ固有）
- `README.md`：ディレクトリ説明（コピー、編集不可）

### docs/adr/
- リポジトリ固有のADR（実体）
- `README.md`：ADR説明（コピー、編集不可）

### docs/governance/
- `docs-style-guide.md`：ドキュメント記述ルール（シンボリックリンク）
- `shared-rules.md`：複数リポジトリ共通ルール（シンボリックリンク）
- `README.md`：ガバナンス説明（コピー、編集不可）

### .claude/
Claude Code設定（シンボリックリンク）：
- `settings.json`：Hooks設定
- `commands/suggest-claude-md.md`
- `skills/`：Agent Skills

### .specify/
GitHub Spec Kit（仕様駆動開発）：
- `memory/constitution.md`：プロジェクトの憲法（実体、カスタマイズ可）
- `specs/`：機能仕様書（実体、リポジトリ固有）
- `.claude/commands/`：speckit-*コマンド（シンボリックリンク）
- `templates/`：仕様書テンプレート（シンボリックリンク）
- `README.md`：Spec Kit説明（コピー、編集不可）

### 同期について
- **シンボリックリンク**: limimeshi-docs更新で自動反映
- **READMEコピー**: limimeshi-docsから`/sync-shared-rules [リポジトリ名]`で同期
- **リポジトリ固有**: constitution.md、roadmap.md、CHANGELOG.md、specs/、adr/
```

## README.mdに追記する内容

```markdown
## 前提条件

このリポジトリは limimeshi-docs と同じ親ディレクトリに配置する必要がある：
\`\`\`
parent-directory/
├── limimeshi-docs/    ← 必須
└── this-repo/
\`\`\`

## ブランチ戦略

このリポジトリは**git-flow**を採用。詳細は [CONTRIBUTING.md](https://github.com/shg25/limimeshi-docs/blob/main/CONTRIBUTING.md) を参照。

| ブランチ | 用途 |
|---------|------|
| `main` | 本番環境 |
| `develop` | 開発環境（通常はここから作業） |
| `feature/*` | 機能開発 |
```

## 注意事項

- 対象リポジトリが存在しない場合はエラー
- 既存ファイルがある場合は上書き前に確認
- constitution.mdは初期値としてコピー後、各リポジトリで独立管理
- セットアップ後はコミットを忘れずに
