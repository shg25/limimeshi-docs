# AI Agent Context: 期間限定めし（リミメシ）

このファイルはClaude CodeなどのAIエージェントがプロジェクトを効率的に理解するための情報です。

## プロジェクト概要

**正式名称**: 期間限定めし（リミメシ）
**英語表記**: Limited Meshi (Limimeshi)
**コンセプト**: なじみのチェーン店で新しい体験を楽しむ

チェーン店の期間限定メニューに特化した情報プラットフォーム。
日本の大手チェーン店（16店舗）の期間限定キャンペーン情報を一覧できるサービス。

## このリポジトリの役割

**limimeshi-docs**: ガバナンス専用リポジトリ（公開リポジトリ）

| 役割 | 説明 |
|------|------|
| **ガバナンス** | プロジェクト全体の企画・方針・ルールを管理 |
| **共通ADR** | 複数リポジトリに影響する技術選定を記録 |
| **マスタードキュメント** | Constitution、WRITING_STYLE_GUIDEのマスターを保持 |

> **Note**: 機能仕様書（specs）やSpec Kitファイルは各実装リポジトリに移行済み

### 実装リポジトリ

| リポジトリ | 役割 | 状態 |
|------------|------|------|
| `limimeshi-admin` | 管理画面（React Admin） | ✅ 実装完了 |
| `limimeshi-android` | Androidアプリ（Kotlin + Jetpack Compose） | 🚧 準備中 |
| `limimeshi-infra` | Firestore Rules/Indexes管理 | 📋 Phase3で作成予定 |
| `limimeshi-web` | Webアプリ | 📋 Phase3で作成予定 |

## 現在のフェーズ

**Phase2: MVP実装**

- ✅ Phase0: 企画の妥当性検証（完了: 2025/10/26）
- ✅ Phase1: 設計・仕様策定（完了: 2025/11/28）
- 🚧 Phase2: MVP実装（limimeshi-admin完了、limimeshi-android準備中）
- 📋 Phase3: ベータリリース（未着手）
- 📋 Phase4: 本番リリース（未着手）

詳細は [roadmap.md](./roadmap.md) を参照

## ディレクトリ構造

```
limimeshi-docs/
├── adr/                 # 共通ADR（複数リポジトリに影響する技術選定）
├── data-model/          # Firestoreスキーマ設計（→Phase3でlimimeshi-infraに移行予定）
├── guides/              # 本番環境セットアップガイド（→Phase3でlimimeshi-infraに移行予定）
├── memory/              # Constitution（憲法）：開発原則のマスター
├── planning/            # Phase0企画ドキュメント（first-idea.md、lean-canvas.md、inception-deck.md）
├── CLAUDE.md            # AI向けプロジェクト情報（このファイル）
├── roadmap.md           # プロジェクト全体のロードマップ
├── WRITING_STYLE_GUIDE.md  # ドキュメント記述ルールのマスター
└── README.md            # ガバナンスリポジトリ説明
```

## 重要なドキュメント

### 必ず参照すべきファイル

1. **[memory/constitution.md](./memory/constitution.md)**
   - **プロジェクトの憲法（最優先）**
   - 開発原則：Test-First、Firebase-First、Manual Operation First、Simplicity など
   - 技術選定方針、品質基準、ガバナンス
   - **全ての実装・設計はこの憲法に準拠する**
   - 新規リポジトリ作成時はこれをコピーして配置

2. **[WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md)**
   - **すべてのドキュメント作成時に必ず従う**
   - 句読点、強調表記、見出しレベル、サービス名表記など

3. **[roadmap.md](./roadmap.md)**
   - プロジェクト全体のロードマップ
   - 各フェーズのタスク一覧と進捗状況

4. **[planning/first-idea.md](./planning/first-idea.md)**
   - サービスの基本設計
   - MVP機能の定義

## ドキュメント記述ルール

新しいドキュメントを作成する際は、**必ず [WRITING_STYLE_GUIDE.md](./WRITING_STYLE_GUIDE.md) を参照**

### 主要ルール（抜粋）

- **句読点**: 箇条書き・説明文では「。」を使わない
- **文体**: 「〜します」ではなく「〜する」
- **サービス名**: タイトル・正式表記は「期間限定めし（リミメシ）」、文中は「リミメシ」も許容
- **強調表記**: サービス名、技術用語の初出、重要なルールのみ
- **Phase表記**: `Phase0`、`Phase1`（スペースなし）
- **コロン**: 全角「：」を使用

## GitHub Spec Kit（仕様駆動開発）

このプロジェクトは **GitHub Spec Kit** を採用

### Spec Kitファイルの配置

Spec Kitファイル（specs/、templates/、.claude/commands/）は**各実装リポジトリに配置**：
- limimeshi-admin: `.specify/specs/001-admin-panel/`
- limimeshi-android: `.specify/specs/002-chain-list/`、`.specify/specs/003-favorites/`

### Spec Kitの哲学

- **Spec-Driven Development**: 仕様が王様、コードは従者
- 仕様書からコードを生成する
- 仕様と実装のギャップをゼロにする

## 新規リポジトリ作成ルール

新しい実装リポジトリを作成する際は以下を実施：

### 1. Spec Kit導入

各実装リポジトリには`.specify/`ディレクトリを作成し、Spec Kit構造を導入

```
.specify/
├── .claude/commands/   # スラッシュコマンド
├── memory/
│   └── constitution.md # ← 本リポジトリのmemory/constitution.mdをコピー
├── specs/              # 機能仕様書
└── templates/          # テンプレート
```

### 2. ドキュメントスタイル統一

- 本リポジトリの WRITING_STYLE_GUIDE.md を参照
- 必要に応じて各リポジトリにコピー

### 3. ADR配置

- **共通ADR**: 本リポジトリ（limimeshi-docs/adr/）
- **固有ADR**: 各実装リポジトリ（docs/adr/）

## 共通ADR一覧

| ADR | 内容 |
|-----|------|
| [ADR-001](./adr/001-use-firebase-for-backend.md) | Use Firebase for backend |
| [ADR-002](./adr/002-adopt-multi-repository-structure.md) | Adopt multi-repository structure |
| [ADR-003](./adr/003-use-react-admin-for-admin-panel.md) | Use React Admin for admin panel |
| [ADR-004](./adr/004-use-manual-data-entry-for-phase2.md) | Use manual data entry for Phase 2 |
| [ADR-005](./adr/005-deploy-using-firebase-hosting-multi-site.md) | Deploy using Firebase Hosting multi-site |

## 技術スタック

- **バックエンド**: Firebase (Auth, Firestore, Functions, Hosting, Scheduler)
- **管理画面**: React Admin（TypeScript + React）
- **Androidアプリ**: Kotlin + Jetpack Compose
- **CI/CD**: GitHub Actions

### データ更新方針

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

**最終更新**: 2025/12/03
