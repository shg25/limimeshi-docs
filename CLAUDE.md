# AI Agent Context: 期間限定めし（リミメシ）

このファイルはClaude CodeなどのAIエージェントがプロジェクトを効率的に理解するための情報です。

## プロジェクト概要

**正式名称**: 期間限定めし（リミメシ）
**英語表記**: Limited Meshi (Limimeshi)
**コンセプト**: なじみのチェーン店で新しい体験を楽しむ

チェーン店の期間限定メニューに特化した情報プラットフォーム。
日本の大手チェーン店（16店舗）の期間限定メニューを一覧できるWebサービス。

## このリポジトリの役割

**limimeshi-docs**: 企画・設計ドキュメント専用リポジトリ（公開リポジトリ）

実装は別リポジトリ:
- `limimeshi-web`: 一般向けWebアプリ
- `limimeshi-admin`: 管理画面
- `limimeshi-jobs`: 定期処理

## 現在のフェーズ

**Phase1: 設計・仕様策定**（進行中）

- ✅ Phase0: 企画の妥当性検証（完了: 2025/10/26）
- 🔄 Phase1: 設計・仕様策定（進行中、GitHub Spec Kit公式手法への適合作業中）
- 📋 Phase2: MVP実装（未着手）
- 🚀 Phase3: ベータリリース（未着手）
- 🚀 Phase4: 本番リリース（未着手）

詳細は [roadmap.md](./roadmap.md) と [MIGRATION_TO_SPEC_KIT.md](./MIGRATION_TO_SPEC_KIT.md) を参照

## ディレクトリ構造

```
limimeshi-docs/
├── .claude/commands/    # GitHub Spec Kit スラッシュコマンド
│   ├── speckit-specify.md     # /speckit-specify: 機能仕様書を生成
│   ├── speckit-plan.md        # /speckit-plan: 実装計画を生成
│   ├── speckit-tasks.md       # /speckit-tasks: タスクリストを生成
│   └── ...                    # その他のコマンド
├── memory/              # 開発原則（Constitution）
│   └── constitution.md        # プロジェクトの憲法
├── scripts/             # GitHub Spec Kit スクリプト
│   └── bash/                  # Bash版スクリプト
├── templates/           # GitHub Spec Kit テンプレート
│   ├── spec-template.md       # 機能仕様書テンプレート
│   ├── plan-template.md       # 実装計画テンプレート
│   └── tasks-template.md      # タスクリストテンプレート
├── planning/            # Phase0企画ドキュメント（完了）
│   ├── first-idea.md          # 基本アイデア、ターゲット、データモデル、技術スタック
│   ├── lean-canvas.md         # リーンキャンバス（9ブロック）
│   └── inception-deck.md      # インセプションデッキ（10の質問）
├── specs/               # 機能設計書（GitHub Spec Kit形式）
│   ├── 001-menu-list/         # メニュー一覧機能
│   ├── 002-favorites/         # お気に入り登録機能
│   └── 003-admin-panel/       # 管理画面機能
├── adr/                 # Architecture Decision Records
├── api/                 # API仕様書
├── data-model/          # データモデル詳細設計
├── roadmap.md           # プロジェクト全体のロードマップ
├── WRITING_STYLE_GUIDE.md  # ドキュメント記述ルール
└── README.md            # プロジェクト概要
```

各ディレクトリの詳細は各 `README.md` を参照

## 重要なドキュメント

### 必ず参照すべきファイル

1. **[memory/constitution.md](./memory/constitution.md)**
   - **プロジェクトの憲法（最優先）**
   - 開発原則：Test-First、Firebase-First、Manual Operation First、Simplicity など
   - 技術選定方針、品質基準、ガバナンス
   - **全ての実装・設計はこの憲法に準拠する**

2. **[WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md)**
   - **すべてのドキュメント作成時に必ず従う**
   - 句読点、強調表記、見出しレベル、サービス名表記など

3. **[roadmap.md](./roadmap.md)**
   - Phase1のタスク一覧
   - 現在の進捗状況

4. **[planning/first-idea.md](./planning/first-idea.md)**
   - サービスの基本設計
   - Phase2（MVP）で実装する機能の定義

## ドキュメント記述ルール（重要）

新しいドキュメントを作成する際は、**必ず [WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md) を参照**してください。

### 主要ルール（抜粋）

- **句読点**: 箇条書き・説明文では「。」を使わない
- **文体**: 「〜します」ではなく「〜する」
- **サービス名**: タイトル・正式表記は「期間限定めし（リミメシ）」、文中は「リミメシ」も許容
- **強調表記**: サービス名、技術用語の初出、重要なルールのみ
- **Phase表記**: `Phase0`、`Phase1`（スペースなし）
- **コロン**: 全角「：」を使用

詳細は [WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md) 参照

## GitHub Spec Kit（仕様駆動開発）

このプロジェクトは **GitHub Spec Kit** を採用（2025/10/27）

### スラッシュコマンド

以下のコマンドを使って仕様書を生成：
- `/speckit-specify <機能の説明>`: 機能仕様書（spec.md）を生成
- `/speckit-plan`: 実装計画（plan.md）を生成
- `/speckit-tasks`: タスクリスト（tasks.md）を生成
- `/speckit-clarify`: 仕様の曖昧な点を明確化
- `/speckit-implement`: 実装開始

### GitHub Spec Kitの哲学

- **Spec-Driven Development**: 仕様が王様、コードは従者
- 仕様書からコードを生成する
- 仕様と実装のギャップをゼロにする

詳細は [MIGRATION_TO_SPEC_KIT.md](./MIGRATION_TO_SPEC_KIT.md) を参照

## Phase1のタスク（現在進行中）

### 1-1. ディレクトリ構成の整備 ✅ 完了
- specs/, adr/, api/, data-model/ を作成済み
- .claude/commands/, scripts/, templates/, memory/ を追加（GitHub Spec Kit）

### 1-2. GitHub Spec Kitで機能設計 🔄 作業中
- メニュー一覧機能 → `specs/001-menu-list/spec.md`
- お気に入り登録機能 → `specs/002-favorites/spec.md`
- 管理画面機能 → `specs/003-admin-panel/spec.md`

### 1-3. ADRで技術選定記録 📋 未着手
- なぜFirebaseを選んだか → `adr/001-why-firebase.md`
- モノレポ vs マルチリポ → `adr/002-monorepo-vs-multirepo.md`
- React Adminを選んだ理由 → `adr/003-react-admin.md`
- データ更新の運用方針 → `adr/004-manual-data-update.md`

### 1-4. データモデル詳細設計 📋 未着手
- Firestoreコレクション設計 → `data-model/firestore-collections.md`
- セキュリティルール設計 → `data-model/security-rules.md`
- インデックス設計 → `data-model/indexes.md`

## AI向けの注意事項

### ドキュメント作成時

1. **WRITING_STYLE_GUIDE.mdを必ず参照**してから作成
2. 適切なディレクトリに配置（specs/, adr/, data-model/など）
3. サービス名は「期間限定めし（リミメシ）」または「リミメシ」
4. Phase2（MVP）の機能リストは [planning/first-idea.md](./planning/first-idea.md) 参照

### 新しい手法・フレームワーク採用時

**重要**：新しい手法やフレームワークを採用した場合、README.mdの「採用している手法・フレームワーク」表に必ず追記する

**追記内容**：
- フェーズ（Phase0、Phase1など）
- 手法・フレームワークの正式名称
- 役割・目的
- 公式リンク（公式サイトまたは公式ドキュメント）

**例**：
```markdown
| Phase1 | Jest | JavaScriptテストフレームワーク | [Jest公式](https://jestjs.io/) |
```

**条件**：
- 業界標準または公式推奨の手法のみ追記
- 独自手法・我流は追記しない（再現性・信頼性を重視）

### 技術スタック

- **バックエンド**: Firebase (Auth, Firestore, Functions, Hosting, Scheduler)
- **フロントエンド**: React, TypeScript
- **管理画面**: React Admin
- **CI/CD**: GitHub Actions

### データ更新方針（重要）

- **スクレイピング禁止**（法的リスク回避）
- **完全手動運用**（Phase2）
- 補助ツール: Googleアラート、RSSリーダー、AI補助（事実情報の抽出）

## 開発方針

- **仕様駆動開発（Spec-Driven Development）**: GitHub Spec Kit採用、仕様が王様
- **ドキュメント駆動開発**: 実装前に企画・設計を詳細に文書化
- **AI活用前提**: Claude Code使用、スラッシュコマンドで効率化
- **Constitution（憲法）遵守**: 全ての実装・設計は憲法に準拠
- **公開リポジトリ**: 透明性を重視
- **個人開発**: 1名（重次弘規）
- **品質最優先**: CI/CD・テスト・負荷テストを最優先（後回し厳禁）

## 成功の定義

- Phase3で50人のベータユーザーから好意的なフィードバック
- Phase4で月間1,000人のアクティブユーザー獲得
- 運用コストを広告収益で賄える状態

---

**最終更新**: 2025/10/27

## Active Technologies
- TypeScript 5.x, React 18.x (001-admin-panel)
- Firestore (NoSQL document database) (001-admin-panel)

## Recent Changes
- 001-admin-panel: Added TypeScript 5.x, React 18.x
