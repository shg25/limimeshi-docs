# コントリビューションガイド

## 基本方針

このプロジェクトは個人開発プロジェクトですが、以下のような形でのコントリビューションは歓迎です！

- 誤字脱字の修正
- ドキュメントの改善提案
- サービスアイデアへのフィードバック
- 技術的なアドバイス

## リポジトリ別ブランチ戦略

このプロジェクトでは、リポジトリの役割に応じて異なるブランチ戦略を採用しています。

| リポジトリ | ブランチ戦略 | 理由 |
|-----------|------------|------|
| limimeshi-docs | GitHub Flow | ドキュメント専用、シンプルで十分 |
| limimeshi-admin | git-flow | 実装リポジトリ、CI/CD・リリース管理が必要 |
| limimeshi-android | git-flow | 実装リポジトリ、CI/CD・リリース管理が必要 |
| limimeshi-infra | git-flow | インフラ管理、本番/開発環境の分離が必要 |

### 新規リポジトリ作成時の指針

- **ドキュメント専用リポジトリ** → GitHub Flow
- **実装リポジトリ（コード）** → git-flow
- **インフラ・設定リポジトリ** → git-flow

各リポジトリのCONTRIBUTING.mdにブランチ戦略の詳細を記載してください。

---

## GitHub Flow（このリポジトリで採用）

このリポジトリ（limimeshi-docs）では**GitHub Flow**を採用しています。

### GitHub Flowとは

シンプルなブランチ戦略で、以下の流れで作業します：

1. **mainブランチは常にデプロイ可能な状態**
2. **新しい作業は必ずブランチを切る**
3. **プルリクエストを作成してレビュー**
4. **マージ後はブランチを削除**

### ワークフロー

```
main ─┬─ feature/add-section ─→ PR ─→ merge ─→ main
      │
      └─ fix/typo ─→ PR ─→ merge ─→ main
```

---

## git-flow（実装リポジトリで採用）

実装リポジトリ（limimeshi-admin、limimeshi-android等）では**git-flow**を採用しています。

### git-flowとは

Vincent Driessen氏が提唱したブランチモデルで、リリース管理とCI/CDに適した戦略です。

### ブランチ構成

| ブランチ | 用途 | マージ先 |
|---------|------|---------|
| `main` | 本番環境（安定版） | - |
| `develop` | 開発環境（次期リリース準備） | main |
| `feature/*` | 機能開発 | develop |
| `release/*` | リリース準備 | main + develop |
| `hotfix/*` | 緊急修正 | main + develop |

### ワークフロー

```
main ────────────────────────────────────→ 本番リリース
  ↑                                    ↑
  │                              release/1.0
  │                                    ↑
develop ─┬─ feature/add-chain-crud ────┘
         │
         └─ feature/add-campaign-list ─→ develop

hotfix/security-fix ─────────────────→ main + develop
```

詳細は各リポジトリのCONTRIBUTING.mdを参照してください。

---

## ブランチ命名規則

ブランチ名は以下の形式で命名してください：

```
<type>/<description>
```

### type（種類）

| type | 用途 | 例 |
|------|------|-----|
| `feature` | 新しいセクション・ドキュメント追加 | `feature/add-lean-canvas` |
| `fix` | 誤字脱字・軽微な修正 | `fix/typo-in-readme` |
| `update` | 既存ドキュメントの更新・改善 | `update/persona-section` |
| `docs` | ドキュメント構造の変更 | `docs/restructure-files` |

### description（説明）

- 英語または日本語のローマ字表記
- ハイフン（`-`）で単語を区切る
- 簡潔で分かりやすく

#### Good

- `feature/add-inception-deck`
- `fix/typo-in-first-idea`
- `update/roadmap-phase2`

#### Bad

- `feature/add_inception_deck`（アンダースコアは使わない）
- `mywork`（何の作業か分からない）
- `fix`（説明が不足）

---

## コミットメッセージ

### 基本フォーマット

```
<type>: <subject>

<body>（任意）
```

### type（種類）

| type | 用途 |
|------|------|
| `feat` | 新機能・新セクション追加 |
| `fix` | バグ修正・誤字脱字修正 |
| `docs` | ドキュメント修正 |
| `style` | フォーマット変更（内容に影響しない） |
| `refactor` | リファクタリング |
| `chore` | その他の雑務 |

### subject（件名）

- 日本語または英語
- 簡潔に（50文字以内推奨）
- 命令形で書く（例：「追加する」ではなく「追加」）

### 例

#### Good

```
feat: リーンキャンバスのセクションを追加

9つのブロックを埋めて、ビジネスモデルを可視化しました。
```

```
fix: README.mdの誤字を修正
```

```
docs: ディレクトリ構成を更新
```

#### Bad

```
update（何を更新したか不明）
```

```
いろいろ修正した（具体性がない）
```

---

## プルリクエスト

### プルリクエストの作成

1. **ブランチを作成**
   ```bash
   git checkout -b feature/add-section
   ```

2. **変更をコミット**
   ```bash
   git add .
   git commit -m "feat: 新しいセクションを追加"
   ```

3. **リモートにプッシュ**
   ```bash
   git push origin feature/add-section
   ```

4. **GitHubでプルリクエストを作成**

### プルリクエストのタイトル

コミットメッセージと同じ形式で記載：

```
feat: リーンキャンバスのセクションを追加
```

### プルリクエストの説明

以下の情報を含めてください

- 変更内容の要約
- なぜこの変更が必要か
- 関連するIssue（あれば）

#### テンプレート例

```markdown
## 変更内容
リーンキャンバスのセクションを追加しました。

## 理由
ビジネスモデルを1ページで可視化するため。

## 関連Issue
#1
```

---

## レビュープロセス

- プルリクエストは作成者以外がレビュー（現状は作成者がセルフレビュー）
- レビュー後、問題がなければ`main`にマージ
- マージ後はブランチを削除

---

## mainブランチの保護

mainブランチは以下のルールで保護されています：

- ✅ 直接pushは禁止
- ✅ プルリクエスト経由でのみマージ可能
- ⚠️ レビュー必須（現状はセルフレビューで可）

---

## ローカルでの作業例

```bash
# 1. 最新のmainブランチを取得
git checkout main
git pull origin main

# 2. 新しいブランチを作成
git checkout -b feature/add-lean-canvas

# 3. ファイルを編集
# （エディタでドキュメントを編集）

# 4. 変更を確認
git status
git diff

# 5. 変更をコミット
git add .
git commit -m "feat: リーンキャンバスを追加"

# 6. リモートにプッシュ
git push origin feature/add-lean-canvas

# 7. GitHubでプルリクエストを作成
# （ブラウザで操作）

# 8. マージ後、ブランチを削除
git checkout main
git pull origin main
git branch -d feature/add-lean-canvas
```

---

## 質問やフィードバック

質問やフィードバックは、以下の方法でお願いします

- Issue：具体的な問題や提案
- Discussion：アイデアや議論

---

ご協力ありがとうございます！
