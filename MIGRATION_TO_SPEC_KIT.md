# GitHub Spec Kit公式手法への適合（軌道修正）

作成日：2025/10/26

---

## 背景と決定理由

### なぜ軌道修正するのか

- Phase1-2で機能設計書（`specs/menu-list.md`、`specs/favorites.md`、`specs/admin-panel.md`）を作成したが、GitHub Spec Kitの公式テンプレートを正しく理解できておらず、一般的な機能仕様書の形式で作成
- 認識のズレが判明したため、以下の理由により公式手法に適合するよう軌道修正することを決定した。

### 決定理由

1. **再現性・信頼性を重視**
   - 独自手法ではなく、業界標準の手法を正しく適用できることを証明
   - GitHubが公式に推奨する最新手法（2025年9月公開）を実践
   - リーンキャンバス → インセプションデッキ → **GitHub Spec Kit** → AI実装という一貫したストーリー

2. **AI駆動開発の経験を重視**
   - このプロジェクトは一貫してClaude CodeなどのAIを使うことを前提としている
   - GitHub Spec KitはClaude Code、GitHub Copilot、Cursor、Gemini CLIなどとの連携が前提
   - 「AIとどう協働するか」という経験は今後のキャリアで間違いなく役立つ

3. **仕様駆動開発（Spec-Driven Development）の哲学に共感**
   - 「コードが王様」ではなく「仕様が王様」
   - 仕様書からコードを生成する
   - 仕様と実装のギャップをゼロにする

4. **タイミングの良さ**
   - GitHub Spec Kitは2025年9月公開（つまり最新）
   - 「最新手法をいち早く実践した」というアピールができる

### 現状の問題点

現在作成した仕様書は：
- ❌ GitHub Spec Kitの公式テンプレートを使っていない
- ❌ ディレクトリ構造が違う（`specs/menu-list.md` ではなく `specs/001-menu-list/spec.md`）
- ❌ スラッシュコマンドを使っていない
- ⚠️ 内容は悪くないが、**公式手法ではない**

---

## GitHub Spec Kitとは

### 概要

**GitHub Spec Kit**は、GitHubが2025年9月に公開した**仕様駆動開発（Spec-Driven Development: SDD）のためのツールキット/テンプレート集**

### 特徴

- **仕様が王様**：仕様書からコードを生成する（コードが仕様に従う）
- **AIエージェントとの連携**：Claude Code、GitHub Copilot、Cursor、Gemini CLIなどと連携
- **スラッシュコマンド**：`/speckit.specify`、`/speckit.plan`、`/speckit.tasks`で15分で完全な仕様書が完成
- **公式テンプレート**：`spec-template.md`、`plan-template.md`、`tasks-template.md`、`constitution.md`
- **ディレクトリ構造**：`specs/001-feature-name/spec.md`、`specs/001-feature-name/plan.md`など

### 公式リポジトリ

- https://github.com/github/spec-kit

### 参考記事

- [GitHubの新ツールspec-kitを実際に動かしてできたことと、注意点など](https://www.taneyats.com/entry/try-spec-kit)
- [GitHub、仕様駆動開発ツールキット「Spec Kit」を紹介](https://gihyo.jp/article/2025/09/github-spec-kit)
- [GitHub製のSpec Kitで仕様駆動開発に入門してみた](https://dev.classmethod.jp/articles/spec-driven-development-with-github-spec-kit/)

---

## 作業手順

### Phase1：環境セットアップ（Claude Codeセッション1）

- [ ] 1-1. スラッシュコマンドのセットアップ
  - [ ] `/tmp/spec-kit/templates/commands/` から `.claude/commands/` にコピー
  - [ ] `/speckit.specify`、`/speckit.plan`、`/speckit.tasks` が使えることを確認

- [ ] 1-2. Constitution（憲法）作成
  - [ ] `memory/constitution.md` に開発原則を定義
  - [ ] Test-First、Library-First、Simplicity、Anti-Abstraction などを記載
  - [ ] プロジェクト固有の原則を追加（例：Firebase優先、手動運用優先、法的リスクゼロなど）

- [ ] 1-3. ディレクトリ構造の変更
  - [ ] 現在の `specs/` を `specs-old/` にリネーム（バックアップ）
  - [ ] 新しい `specs/` ディレクトリを作成
  - [ ] `specs/001-menu-list/`、`specs/002-favorites/`、`specs/003-admin-panel/` を作成

- [ ] 1-4. CLAUDE.mdとREADME.mdの更新
  - [ ] CLAUDE.mdに「GitHub Spec Kitを使用」と明記
  - [ ] スラッシュコマンドの使い方を記載
  - [ ] Constitution（憲法）の参照方法を記載
  - [ ] README.mdにGitHub Spec Kit採用を記載

### Phase2：仕様書の公式テンプレート適合（Claude Codeセッション2以降）

- [ ] 2-1. メニュー一覧機能（001-menu-list）
  - [ ] `/speckit.specify` でspec.mdを生成
  - [ ] `/speckit.plan` でplan.mdを生成
  - [ ] `/speckit.tasks` でtasks.mdを生成
  - [ ] 必要に応じて `data-model.md`、`contracts/` を作成

- [ ] 2-2. お気に入り登録機能（002-favorites）
  - [ ] `/speckit.specify` でspec.mdを生成
  - [ ] `/speckit.plan` でplan.mdを生成
  - [ ] `/speckit.tasks` でtasks.mdを生成

- [ ] 2-3. 管理画面機能（003-admin-panel）
  - [ ] `/speckit.specify` でspec.mdを生成
  - [ ] `/speckit.plan` でplan.mdを生成
  - [ ] `/speckit.tasks` でtasks.mdを生成

- [ ] 2-4. 旧仕様書の削除
  - [ ] `specs-old/` を削除（内容は新仕様書に反映済み）

### Phase3：ADR作成（Phase1-3）

- [ ] 3-1. ADR作成
  - [ ] `adr/001-why-firebase.md`
  - [ ] `adr/002-monorepo-vs-multirepo.md`
  - [ ] `adr/003-react-admin.md`
  - [ ] `adr/004-manual-data-update.md`

### Phase4：データモデル詳細設計（Phase1-4）

- [ ] 4-1. データモデル詳細設計
  - [ ] `data-model/firestore-collections.md`
  - [ ] `data-model/security-rules.md`
  - [ ] `data-model/indexes.md`

---

## 期待される効果

### 就活でのアピールポイント

1. **最新手法を主体的に採用**
   - 「Phase1-2で独自形式を作成したが、より良い手法を発見し、主体的に移行を決断した」
   - 「GitHub公式の最新手法（2025年9月公開）をいち早く実践した」

2. **一貫した開発手法**
   - リーンキャンバス → インセプションデッキ → **GitHub Spec Kit** → AI実装
   - 企画から実装まで、業界標準の手法を一貫して適用

3. **AI駆動開発の実践**
   - Claude Code、GitHub Copilot、Cursorなどとの協働
   - 「仕様駆動開発」という新しい開発パラダイムの経験

4. **ドキュメント重視の姿勢**
   - 仕様書が王様、コードは従者
   - 仕様と実装のギャップをゼロにする設計

### 技術的なメリット

1. **仕様と実装の一貫性**
   - 仕様書からコードを生成するため、仕様と実装が乖離しない
   - 仕様書を更新すると、コードも自動的に更新される

2. **AI可読性の向上**
   - Claude Codeが仕様書を読み取りやすい形式
   - Phase2実装時にClaude Codeがスムーズにコード生成できる

3. **テスト駆動開発の強制**
   - Constitution（憲法）でTest-Firstを強制
   - テストを先に書く文化が自然に身につく

4. **シンプルさの維持**
   - Constitution（憲法）でSimplicity、Anti-Abstractionを強制
   - 過度な抽象化を防ぐ

---

## 変更の影響範囲

### 影響を受けるファイル

#### 削除または移動
- `specs/menu-list.md` → `specs-old/menu-list.md`（バックアップ）
- `specs/favorites.md` → `specs-old/favorites.md`（バックアップ）
- `specs/admin-panel.md` → `specs-old/admin-panel.md`（バックアップ）
- `specs/README.md` → 更新（GitHub Spec Kit形式の説明）

#### 新規作成
- `.claude/commands/speckit-*.md`（スラッシュコマンド）
- `memory/constitution.md`（開発原則）
- `specs/001-menu-list/spec.md`
- `specs/001-menu-list/plan.md`
- `specs/001-menu-list/tasks.md`
- `specs/002-favorites/spec.md`
- `specs/002-favorites/plan.md`
- `specs/002-favorites/tasks.md`
- `specs/003-admin-panel/spec.md`
- `specs/003-admin-panel/plan.md`
- `specs/003-admin-panel/tasks.md`

#### 更新
- `CLAUDE.md`（GitHub Spec Kit使用を明記）
- `README.md`（GitHub Spec Kit採用を記載）
- `roadmap.md`（Phase1-2の方針転換を記録）

### 影響を受けないファイル

- `planning/`（Phase0ドキュメント）
- `adr/`（ADRは後で作成）
- `data-model/`（データモデルは後で作成）
- `api/`（API仕様書は後で作成）
- `WRITING_STYLE_GUIDE.md`（記述ルールは維持）

---

## リスクと対策

### リスク1：コンテキスト切れ

Claude Codeセッション中にコンテキスト切れが発生し、移行手順が中断される

#### 対策

- このドキュメント（MIGRATION_TO_SPEC_KIT.md）にチェックリストを残す
- roadmap.mdに簡潔な記録を残す
- CLAUDE.mdに「現在公式手法への適合作業中」と明記

### リスク2：仕様書の品質低下

公式テンプレートに合わせて書き直すことで、既存の良い部分が失われる

#### 対策

- 旧仕様書を`specs-old/`にバックアップ
- 旧仕様書の内容を参照しながら、公式テンプレートに転記
- ユーザーストーリー、受入基準は既存のものを活用

### リスク3：作業に時間がかかる

適合作業に時間がかかり、Phase1が遅延する

#### 対策

- スラッシュコマンドを活用し、15分で仕様書を生成
- Phase1-2の内容は既にあるため、転記作業が中心
- Phase1-3、Phase1-4は元々未着手のため、影響なし

---

## 今後のスケジュール

### 適合作業（Phase1-2のやり直し）

- 所要時間：2〜3時間（Claude Codeセッション1〜2回）
- 完了条件：3つの機能の仕様書（spec.md、plan.md、tasks.md）が完成

### Phase1-3、Phase1-4（予定通り）

- Phase1-3：ADR作成（4つ）
- Phase1-4：データモデル詳細設計（3つ）

### Phase1完了目標

- 目標日：2025/11/02（1週間以内）
- 条件：Phase1-2〜Phase1-4が全て完了

---

## 参考資料

### GitHub Spec Kit公式

- [GitHub Spec Kit公式リポジトリ](https://github.com/github/spec-kit)
- [spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md)（仕様駆動開発の哲学）
- [spec-template.md](https://github.com/github/spec-kit/blob/main/templates/spec-template.md)（機能仕様書テンプレート）
- [plan-template.md](https://github.com/github/spec-kit/blob/main/templates/plan-template.md)（実装計画テンプレート）
- [tasks-template.md](https://github.com/github/spec-kit/blob/main/templates/tasks-template.md)（タスクリストテンプレート）

### 解説記事

- [GitHubの新ツールspec-kitを実際に動かしてできたことと、注意点など](https://www.taneyats.com/entry/try-spec-kit)
- [GitHub、仕様駆動開発ツールキット「Spec Kit」を紹介](https://gihyo.jp/article/2025/09/github-spec-kit)
- [GitHub製のSpec Kitで仕様駆動開発に入門してみた](https://dev.classmethod.jp/articles/spec-driven-development-with-github-spec-kit/)
- [【Spec Kit】超簡単にスペック駆動開発を実践しよう！](https://aadojo.alterbooth.com/entry/2025/09/06/020000)

---

## 更新履歴

- 2025/10/26：初版作成（公式手法への適合計画）
