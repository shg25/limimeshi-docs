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
- 🔄 Phase1: 設計・仕様策定（進行中）
- 📋 Phase2: MVP実装（未着手）
- 🚀 Phase3: ベータリリース（未着手）
- 🚀 Phase4: 本番リリース（未着手）

詳細は [roadmap.md](./roadmap.md) を参照

## ディレクトリ構造

```
limimeshi-docs/
├── planning/            # Phase0企画ドキュメント（完了）
│   ├── first-idea.md          # 基本アイデア、ターゲット、データモデル、技術スタック
│   ├── lean-canvas.md         # リーンキャンバス（9ブロック）
│   └── inception-deck.md      # インセプションデッキ（10の質問）
├── specs/               # 機能設計書（GitHub Spec Kit形式）
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

1. **[WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md)**
   - **すべてのドキュメント作成時に必ず従う**
   - 句読点、強調表記、見出しレベル、サービス名表記など

2. **[roadmap.md](./roadmap.md)**
   - Phase1のタスク一覧
   - 現在の進捗状況

3. **[planning/first-idea.md](./planning/first-idea.md)**
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

## Phase1のタスク（現在進行中）

### 1-1. ディレクトリ構成の整備 ✅ 完了
- specs/, adr/, api/, data-model/ を作成済み

### 1-2. GitHub Spec Kitで機能設計
- メニュー一覧機能 → `specs/menu-list.md`
- お気に入り登録機能 → `specs/favorites.md`
- 管理画面機能 → `specs/admin-panel.md`

### 1-3. ADRで技術選定記録
- なぜFirebaseを選んだか → `adr/001-why-firebase.md`
- モノレポ vs マルチリポ → `adr/002-monorepo-vs-multirepo.md`
- React Adminを選んだ理由 → `adr/003-react-admin.md`
- データ更新の運用方針 → `adr/004-manual-data-update.md`

### 1-4. データモデル詳細設計
- Firestoreコレクション設計 → `data-model/firestore-collections.md`
- セキュリティルール設計 → `data-model/security-rules.md`
- インデックス設計 → `data-model/indexes.md`

## AI向けの注意事項

### ドキュメント作成時

1. **WRITING_STYLE_GUIDE.mdを必ず参照**してから作成
2. 適切なディレクトリに配置（specs/, adr/, data-model/など）
3. サービス名は「期間限定めし（リミメシ）」または「リミメシ」
4. Phase2（MVP）の機能リストは [planning/first-idea.md](./planning/first-idea.md) 参照

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

- **ドキュメント駆動開発**: 実装前に企画・設計を詳細に文書化
- **AI活用前提**: Claude Code使用
- **公開リポジトリ**: 透明性を重視
- **個人開発**: 1名（重次弘規）
- **品質最優先**: CI/CD・テスト・負荷テストを最優先（後回し厳禁）

## 成功の定義

- Phase3で50人のベータユーザーから好意的なフィードバック
- Phase4で月間1,000人のアクティブユーザー獲得
- 運用コストを広告収益で賄える状態

---

**最終更新**: 2025/10/26
