# プロジェクト進行ロードマップ

作成日：2025/10/13

---

## 全体の流れ

```
Phase0：企画の妥当性検証
  ↓
Phase1：設計・仕様策定
  ↓
Phase2：MVP実装
  ↓
Phase3：ベータリリース・フィードバック収集
  ↓
Phase4：本番リリース・運用
  ↓
Phase5以降：スケールアップ・機能拡張
```

---

## Phase0：企画の妥当性検証

### 0-1. `first-idea.md` の拡充 ✅ 完了

#### 目的
サービスの全体像を多角的に整理する

#### 成果物
`first-idea.md` に以下のセクションを追加
- [x] ターゲットユーザー・ペルソナ ✅
- [x] データモデル（概要）✅
- [x] スクレイピング・データ更新の方針 ✅
- [x] 非機能要件 ✅
- [x] 開発フェーズ・マイルストーン ✅
- [x] リスクと対策 ✅
- [x] モニタリング・分析 ✅

#### 進捗
7/7 完了（100%）

### 0-2. リーンキャンバス作成 ✅ 完了

#### 目的
ビジネスモデル全体を1枚にまとめて、収益性や市場性を検証する

#### 成果物
`lean-canvas.md`

#### 内容
以下の9ブロックを埋める
1. 課題（Problem）
2. 顧客セグメント（Customer Segments）
3. 独自の価値提案（Unique Value Proposition）
4. ソリューション（Solution）
5. チャネル（Channels）
6. 収益の流れ（Revenue Streams）
7. コスト構造（Cost Structure）
8. 主要指標（Key Metrics）
9. 圧倒的な優位性（Unfair Advantage）

#### 進捗
9/9 完了（100%）

### 0-3. インセプションデッキ作成 ✅ 完了

#### 目的
プロジェクトの目的、優先順位、トレードオフを明確化する

#### 成果物
`inception-deck.md`

#### 内容
10の質問に答える
1. 我々はなぜここにいるのか？
2. エレベーターピッチを作る
3. パッケージデザインを作る
4. やらないことリストを作る
5. 「ご近所さん」を探す
6. 解決案を描く
7. 夜も眠れない問題は何だろう？
8. 期間を見極める
9. 何を諦めるのか？
10. 何がどれだけ必要か？

#### 進捗
10/10 完了（100%）

### 0-4. 整合性チェック ✅ 完了

#### 目的
3つのドキュメント間の矛盾を解消し、企画の精度を高める

#### 作業内容
- `first-idea.md`、`lean-canvas.md`、`inception-deck.md` を見比べる
- 矛盾や曖昧な点を発見・修正
- 必要に応じて各ドキュメントを更新

#### 修正内容
1. first-idea.mdから「差分検知システム」を削除（スクレイピング廃止に伴い不要）
2. lean-canvas.mdのKPI目標値を現実的な数値に修正
   - Phase3：MAU 50〜100人、DAU 10〜20人
   - Phase4：MAU 1,000〜3,000人、DAU 100〜300人
   - Phase5以降：MAU 10,000人、DAU 1,000人

---

## Phase1：設計・仕様策定 ✅ 完了

### 1-1. ディレクトリ構成の整備 ✅ 完了

Phase0完了後、以下のディレクトリ構成に整理しました：

```
limimeshi-docs/
├── README.md
├── CLAUDE.md            # AI向けプロジェクト情報
├── WRITING_STYLE_GUIDE.md
├── CONTRIBUTING.md
├── .gitignore
├── roadmap.md
├── planning/            # 企画ドキュメント（Phase0で作成したファイルを移動）
│   ├── README.md
│   ├── first-idea.md
│   ├── lean-canvas.md
│   └── inception-deck.md
├── specs/               # 機能設計書（Phase1で追加）
│   └── README.md
├── adr/                 # Architecture Decision Records（Phase1で追加）
│   └── README.md
├── api/                 # API仕様書（Phase1で追加）
│   └── README.md
└── data-model/          # データモデル詳細（Phase1で追加）
    └── README.md
```

#### 完了内容
- Phase0ドキュメント（first-idea.md、lean-canvas.md、inception-deck.md）をplanning/に移動
- 4つの新規ディレクトリ作成（specs/、adr/、api/、data-model/）
- 各ディレクトリにREADME.md作成（役割と作成予定ドキュメントを記載）
- CLAUDE.md作成（AI向けプロジェクト情報、WRITING_STYLE_GUIDE.mdへのリンク含む）

---

### 重要な軌道修正：GitHub Spec Kit公式手法への適合

#### 日付

2025/10/26

#### 背景

- Phase1-2で機能設計書を作成する際、GitHub Spec Kitの公式テンプレートを正しく理解できておらず、一般的な機能仕様書の形式で作成してしまった
- 認識のズレが判明したため、公式手法に適合するよう軌道修正することを決定

#### 理由

- 業界標準の手法を正しく適用できることを証明
- GitHubが公式に推奨する最新手法（2025年9月公開）を実践
- AI駆動開発（Claude Code、GitHub Copilot、Cursor）との連携を前提とした開発手法
- リーンキャンバス → インセプションデッキ → **GitHub Spec Kit** → AI実装という一貫したストーリー

#### 影響範囲

- Phase1-2の機能設計書を公式テンプレートで書き直し
- ディレクトリ構造を変更（`specs/menu-list.md` → `specs/001-menu-list/spec.md`）
- スラッシュコマンド（`/speckit.specify`、`/speckit.plan`、`/speckit.tasks`）のセットアップ
- Constitution（開発原則）の定義

#### 詳細

[MIGRATION_TO_SPEC_KIT.md](./MIGRATION_TO_SPEC_KIT.md)参照

---

### 1-2. GitHub Spec Kitで機能設計（公式手順） ✅ 完了

**出典**：[GitHub Spec Kit公式リポジトリ](https://github.com/github/spec-kit)

#### 公式ワークフロー

1. `/speckit-specify <機能の説明>` → `spec.md`（機能仕様書）を生成
2. `/speckit-clarify` → 仕様の曖昧な点を明確化（必要に応じて）
3. `/speckit-plan` → `plan.md`（実装計画）を生成
4. `/speckit-tasks` → `tasks.md`（タスクリスト）を生成

#### 対象機能

Phase2 MVP機能（実装優先順位順）：
- ✅ `specs/001-admin-panel/`：管理画面機能（最優先、手動データ登録の基盤）
  - spec.md完成（2025/11/14）
  - タグ管理・入力補助機能をOut of Scopeに移動
  - ふりがな、X Post URL、ステータス自動判定など詳細要件確定
  - 002との整合性確保（ステータス表示ロジック統一）
- ✅ `specs/002-menu-list/`：メニュー一覧機能
  - spec.md完成（2025/11/16）
  - ✅ Phase 0: Research完了（2025/11/19）
  - ✅ Phase 1: Design & Contracts完了（2025/11/19）
  - ✅ tasks.md生成完了（2025/11/19、37タスク）
  - X Post埋め込み表示をPhase2 MVPに含める（商品画像代わり）
  - チェーン店別フィルタリングをOut of Scopeに移動
  - フィルタ選択の永続化、1年経過メニューの自動非表示ルールを追加
  - ステータス表示ロジックを「予定/発売から◯日経過/販売終了」に統一（「販売中」削除）
- ✅ `specs/003-favorites/`：お気に入り登録機能
  - spec.md完成（2025/11/16）
  - ✅ Phase 0: Research完了（2025/11/19）
  - ✅ Phase 1: Design & Contracts完了（2025/11/19）
  - ✅ tasks.md生成完了（2025/11/19、28タスク）
  - Phase2ではログイン必須（未ログインユーザー向けはPhase3以降）
  - お気に入り登録数を集約データとして管理（パフォーマンス最適化）
  - 002のメニュー一覧フィルタと連携
  - 依存関係を明確化（002は003に依存、003は002に依存しない）

**MVP対象外**：
- レビュー機能：Phase3以降に延期（モデレーション・スパム対策の運用負荷を考慮）

#### 完了条件

各機能ごとに `spec.md`、`plan.md`、`tasks.md` が完成 ✅ **達成**

#### 作業順序の調整（2025/11/16）

**変更内容**:
- 当初予定: 001→002→003の順で全て完了してから、ADR作成
- 変更後: **001完了 → ADR-001〜004作成 → 002, 003完了**

**理由**:
1. **ADRの依存関係**: ADR-005（Firebase Hosting構成）を作成した際、ADR-001（Firebase選定）とADR-002（マルチリポ選定）を前提としていることが判明
2. **論理的整合性**: 002, 003はlimimeshi-web（別リポジトリ）に実装するが、その判断根拠（ADR-002: マルチリポ選定）が未記録
3. **基本方針の明文化**: 全機能の前提となる技術選定（Firebase, マルチリポ, React Admin, 手動運用）を先に文書化

**実施順序**:
1. ✅ 001-admin-panel: spec.md, plan.md, research.md, data-model.md, contracts/, quickstart.md完成
2. ✅ ADR-005: Firebase Hosting構成完成
3. ✅ ADR-001〜004作成（基本方針の明文化）
4. ✅ 002-menu-list: Phase 0, Phase 1, tasks.md生成完了
5. ✅ 003-favorites: Phase 0, Phase 1, tasks.md生成完了

---

### 重要な設計判断：リポジトリアーキテクチャの見直し

#### 日付

2025/11/19

#### 背景

Phase1-2でGitHub Spec Kit形式の機能設計を進める中で、以下の不自然さに気づいた：
- limimeshi-docsに001-admin-panel（管理画面）、002-menu-list（Web）、003-favorites（Web）が混在
- 実装先リポジトリが異なる機能の仕様が1つのリポジトリに集約されている
- GitHub Spec Kitの公式構造（`.specify/`）は実装リポジトリと同居することを前提としている

#### 判断内容

**現在の構成**（Phase1完了まで維持）:
```
limimeshi-docs/
├── .claude/commands/      # Spec Kitスラッシュコマンド
├── memory/               # Constitution
├── templates/            # Spec Kitテンプレート
├── specs/                # 全機能の仕様書
│   ├── 001-admin-panel/
│   ├── 002-menu-list/
│   └── 003-favorites/
├── planning/             # Phase0企画ドキュメント
├── adr/                  # ADR
└── ...
```

**Phase2以降の正しい構成**（実装リポジトリ作成時に移行）:
```
limimeshi-docs/              # ガバナンス専用リポジトリ
├── planning/               # Phase0企画ドキュメント
├── adr/                    # プロジェクト全体のADR
└── memory/
    └── constitution.md     # プロジェクト憲法（全リポジトリで共有）

limimeshi-admin/            # 管理画面リポジトリ
├── .specify/
│   ├── specs/001-admin-panel/
│   ├── memory/constitution.md  # シンボリックリンク
│   ├── templates/
│   └── .claude/commands/
└── src/

limimeshi-web/              # Webアプリリポジトリ
├── .specify/
│   ├── specs/
│   │   ├── 002-menu-list/
│   │   └── 003-favorites/
│   ├── memory/constitution.md  # シンボリックリンク
│   ├── templates/
│   └── .claude/commands/
└── src/
```

#### 理由

1. **GitHub Spec Kit公式の意図**：`.specify/`ディレクトリは実装コードと同居させることで、仕様と実装のギャップをゼロにする
2. **リポジトリの責務分離**：limimeshi-docsはガバナンス（憲法、ADR、企画）のみ、実装仕様は各実装リポジトリで管理
3. **Constitution共有**：`memory/constitution.md`はシンボリックリンクで全リポジトリから参照
4. **作業の流れ**：Phase1は企画・設計フェーズなので、現在の構成で完了させてから移行

#### 移行タイミング

- **Phase1完了まで**：現在の構成を維持（limimeshi-docsに全仕様書を集約）
- **Phase2開始時**：実装リポジトリ作成と同時に、各仕様書を適切なリポジトリに移行
  1. limimeshi-adminリポジトリ作成 → 001-admin-panelを移行
  2. limimeshi-webリポジトリ作成 → 002-menu-list、003-favoritesを移行
  3. limimeshi-docsをガバナンス専用に整理

#### 影響

- Phase1の作業には影響なし（現在の構成で完了可能）
- Phase2で実装リポジトリを作成する際に、仕様書も同時に移行
- 移行後は各リポジトリで独立して仕様駆動開発を実施

---

### 1-3. 技術選定記録（ADR）✅ 完了

**出典**：[ADR (Architecture Decision Records)](https://adr.github.io/) - 業界標準の手法

**位置づけ**：GitHub Spec Kit公式手順を補完する業界標準プラクティス

#### 目的

技術選定の理由・背景・トレードオフを記録し、後から判断根拠を追跡可能にする

#### ADR形式

**Michael Nygard形式**（2011年、業界標準）を採用：
- Status（提案/承認/非推奨/置換済み）
- Context（なぜこの決定が必要か）
- Decision（何を選択したか）
- Consequences（この決定がもたらす影響、良い面も悪い面も）

**タイトル命名規則**: Present tense imperative verb phrase（現在形の命令形動詞句）
- 例: "Use Firebase for backend", "Adopt multi-repository structure"

#### 作成済みADR

1. ✅ `adr/001-use-firebase-for-backend.md`：Firebase選定理由
   - モバイルアプリ統合機能（Crashlytics、App Distribution、Cloud Messaging）を追加
   - Firebase Hosting vs App Hostingの違いを明記
2. ✅ `adr/002-adopt-multi-repository-structure.md`：マルチリポジトリ選定
   - 公開リポジトリ方針をREADME.mdに分離（技術選定と運用方針を分離）
3. ✅ `adr/003-use-react-admin-for-admin-panel.md`：React Admin選定
4. ✅ `adr/004-use-manual-data-entry-for-phase2.md`：完全手動運用の選定
   - 差分検知システムは Constitution V. Legal Risk Zero により取りやめ
5. ✅ `adr/005-deploy-using-firebase-hosting-multi-site.md`：Firebase Hosting構成
   - Phase3移行手順をroadmap.mdに移動（ADRは決定記録に専念）

#### 整合性確保

- Constitution（memory/constitution.md）との整合性確認完了
- first-idea.mdとの矛盾解消完了
  - 差分検知システムの取りやめを明記
  - レビュー機能のPhase3延期を明記
  - limimeshi-infraリポジトリを追記

---

### 1-4. Firestoreデータベース設計 ✅ 完了

**出典**：[Firestore公式ベストプラクティス](https://firebase.google.com/docs/firestore/best-practices)

**位置づけ**：GitHub Spec Kit公式手順を補完する、Firebase公式推奨の設計手法

#### 目的

アクセスパターンベースでFirestoreのコレクション構造、セキュリティルール、インデックスを設計

#### 成果物

- ✅ `data-model/firestore-best-practices.md`：Firestore公式ベストプラクティスまとめ
- ✅ `data-model/firestore-collections.md`：コレクション設計（アクセスパターン、非正規化戦略）
- ✅ `data-model/security-rules.md`：セキュリティルール設計
- ✅ `data-model/indexes.md`：複合インデックス設計

#### 参考

- [Firestoreデータモデルの選択](https://firebase.google.com/docs/firestore/data-model)
- [セキュリティルールのベストプラクティス](https://firebase.google.com/docs/firestore/security/rules-structure)

---

## Phase2：MVP実装 🚧 未着手

### 2-0. リポジトリ作成と仕様書移行

**前提**：Phase1で作成した仕様書（specs/）を各実装リポジトリに移行

#### タスク

1. **limimeshi-adminリポジトリ作成**
   - [ ] リポジトリ作成（GitHub）
   - [ ] `.specify/`ディレクトリ構成作成（specs/, memory/, templates/, .claude/commands/）
   - [ ] `specs/001-admin-panel/`を移行（limimeshi-docsから）
   - [ ] `memory/constitution.md`をシンボリックリンクで参照
   - [ ] templates/、.claude/commands/をコピー

2. **limimeshi-webリポジトリ作成**
   - [ ] リポジトリ作成（GitHub）
   - [ ] `.specify/`ディレクトリ構成作成
   - [ ] `specs/002-menu-list/`を移行（limimeshi-docsから）
   - [ ] `specs/003-favorites/`を移行（limimeshi-docsから）
   - [ ] `memory/constitution.md`をシンボリックリンクで参照
   - [ ] templates/、.claude/commands/をコピー

3. **limimeshi-docsの整理**
   - [ ] specs/ディレクトリを削除（全仕様書を移行済み）
   - [ ] .claude/commands/、templates/を削除（各実装リポジトリにコピー済み）
   - [ ] README.md更新（ガバナンス専用リポジトリであることを明記）
   - [ ] planning/、adr/、memory/のみ残す

#### 移行後の構成

- **limimeshi-docs**：ガバナンス専用（planning/, adr/, memory/constitution.md）
- **limimeshi-admin**：管理画面の実装と仕様（.specify/specs/001-admin-panel/）
- **limimeshi-web**：Webアプリの実装と仕様（.specify/specs/002-menu-list/, 003-favorites/）

---

### 2-1. リポジトリセットアップ

#### 対象リポジトリ
- `limimeshi-web`（仕様書移行後に初期化）
- `limimeshi-admin`（仕様書移行後に初期化）
- `limimeshi-jobs`（スケジューラ実行用）
- `limimeshi-infra`（Firestoreルール・インデックス管理）

### 2-2. MVP開発

#### 初期リリース対象
- 16チェーン店（`first-idea.md` 参照）
- Phase2（MVP）必須機能
  - メニュー一覧表示
  - お気に入り登録
  - 管理画面（入力補助機能含む）
- 運用効率化の補助ツール
  - Googleアラート
  - RSSリーダー
  - AI補助（事実情報の抽出）
- レビュー機能と位置情報機能はPhase3以降
- スクレイピングは行わず、完全手動運用（法的リスク回避）

### 2-3. テスト・品質保証

---

## Phase3：ベータリリース・フィードバック収集 🚀 未着手

### 3-0. インフラ分離（limimeshi-web 作成前に実施）

**詳細**: [ADR-005](./adr/005-firebase-hosting-multi-site.md) 参照

#### タスク
- [ ] limimeshi-infra リポジトリ作成
- [ ] firestore.rules/indexes を limimeshi-admin から移行
- [ ] デプロイスクリプト作成（deploy-dev.sh, deploy-prod.sh）
- [ ] limimeshi-admin の firebase.json から firestore 設定を削除
- [ ] 移行確認（開発環境でテスト）

**重要**: このタスクは limimeshi-web リポジトリ作成の**直前**に実施

---

### 3-1. ベータリリース
### 3-2. フィードバック収集
### 3-3. 改善実施

---

## Phase4：本番リリース・運用 🚀 未着手

### 4-1. 本番リリース
### 4-2. マーケティング
### 4-3. モニタリング・分析
### 4-4. 継続的改善

---

## Phase5以降：スケールアップ・機能拡張 🚀 未着手

### 5-1. レビュー機能の実装
### 5-2. 位置情報機能の実装
### 5-3. モバイルアプリ展開
### 5-4. チェーン店との提携

---

## 現在の状況

| フェーズ | ステータス | 進捗率 |
|---------|-----------|--------|
| Phase0：企画の妥当性検証 | ✅ 完了 | 100% |
| Phase1：設計・仕様策定 | ✅ 完了 | 100% |
| Phase2：MVP実装 | 🚧 未着手 | 0% |
| Phase3：ベータリリース | 🚀 未着手 | 0% |
| Phase4：本番リリース・運用 | 🚀 未着手 | 0% |
| Phase5以降：スケールアップ | 🌟 未着手 | 0% |

---

## 完了したタスク

### Phase0-1：`first-idea.md` の拡充
- ✅ ターゲットユーザー・ペルソナ
  - 4つのペルソナを作成（堅実派会社員、中高生、営業マン、OL）
  - サービスのコアバリュー「なじみのチェーン店で新しい体験を楽しむ」を定義
  - 競合サービスとの差別化を明確化
- ✅ データモデル（概要）
  - 7つのエンティティを定義（Tags、Chains、Menus、Users、Admins、Reviews、Favorites）
  - タグの正規化設計
  - 認証・権限の分離設計（UsersとAdminsのコレクション分け）
  - UI上の工夫（食べた回数バッジ表示）
- ✅ データ更新の方針
  - スクレイピングは行わず、完全手動運用（利用規約完全遵守）
  - 運用効率化の補助ツール（Googleアラート、RSSリーダー、AI補助）
  - 法的リスクをゼロにする設計
  - Phase3以降はチェーン店提携による半自動化を目指す
- ✅ 非機能要件
  - ISO/IEC 25010:2023に基づいて10の品質特性を定義
  - 各品質特性に「なぜ重要か」「測定指標」「現時点での方針」「TBD」を記載
  - 測定指標には具体例を追加（サービス固有の文脈に落とし込み）
  - セキュリティではプライバシー保護を強化（2023版で追加）
  - AIシステム品質、環境サステナビリティを含む最新標準に対応
- ✅ 開発フェーズ・マイルストーン
  - Phase0〜Phase5の6つのフェーズを定義
  - 各フェーズに「目的」「主要タスク」「成果物」「完了条件」を記載
  - roadmap.mdとの整合性を確保
  - Phase0は企画検証、Phase1は設計、Phase2はMVP実装、Phase3はベータ、Phase4は本番運用、Phase5以降は拡張
- ✅ リスクと対策
  - 7つの主要リスクを整理（法的リスク、データ鮮度、ユーザー獲得、技術的負債、提携交渉、コスト爆発、セキュリティ）
  - 各リスクに「リスク内容」「影響度」「対策」「現状」を記載
  - 特にコスト爆発リスクを詳細化（予算管理、技術的対策、悪意あるユーザー対策）
- ✅ モニタリング・分析
  - 7つの監視領域を定義（アクセス解析、ユーザー行動分析、エラー監視、パフォーマンス監視、コスト監視、KPI、セキュリティ監視）
  - 各領域に「測定指標」「ツール」「実装時期」「現時点での方針」を記載
  - Firebase中心のモニタリング体制（GA4、Firebase Analytics、Sentry検討）
  - コスト監視を重要視し予算アラート設定を明記
- ✅ 機能の優先順位
  - Phase2（MVP）必須機能を明確化（メニュー一覧、お気に入り、管理画面）
  - Phase3以降の機能を定義（レビュー、位置情報、販売終了報告）
  - レビュー機能はPhase3に延期し、運用安定化を最優先
- ✅ OS・ブラウザのサポート範囲
  - シンプルで実用的な方針を策定
  - モダンブラウザ最新版（Chrome、Safari）、iOS/Android直近3世代程度
  - 個人開発に適した柔軟な表現で、メンテナンスフリーな記述

### Phase0-2：`lean-canvas.md` の作成
- ✅ リーンキャンバス（9ブロック）
  - 課題：情報が散在、期間限定を見逃す、選択肢が広がらない
  - 顧客セグメント：30〜50代、節約志向、チェーン店利用者
  - 独自の価値提案：「なじみのチェーン店で新しい体験を楽しむ」
  - ソリューション：メニュー一覧、お気に入り、レビュー機能
  - チャネル：SEO、SNS、外部AI、口コミ
  - 収益の流れ：SSP広告、チェーン店からの告知広告
  - コスト構造：Firebase月0〜5,000円、運用工数1日30分〜1時間
  - 主要指標：MAU、DAU、レビュー投稿数、広告収益
  - 圧倒的な優位性：独自ポジショニング、手動運用の信頼性、提携可能性
- ✅ リーンキャンバスの全体像を1枚の表で表現
- ✅ 検証すべき仮説をリスクが高い順に整理（ユーザー獲得、運用負荷、収益化）

### Phase0-3：`inception-deck.md` の作成
- ✅ インセプションデッキ（10の質問）
  - 1. 我々はなぜここにいるのか？：チェーン店の期間限定メニューを見逃さない仕組みを作る
  - 2. エレベーターピッチ：「食べログ」の「期間限定メニュー特化版」、「TVer」の「チェーン店メニュー版」
  - 3. パッケージデザイン：「なじみのチェーン店で新しい体験を楽しむ」
  - 4. やらないことリスト：Phase2でレビュー・位置情報・通知機能を延期、スクレイピング廃止
  - 5. 「ご近所さん」を探す：食べログ、Retty、各チェーン公式アプリ、Google Maps、SNS
  - 6. 解決案を描く：ユーザージャーニー、システムアーキテクチャを整理
  - 7. 夜も眠れない問題：ユーザー獲得、運用負荷、コスト爆発、法的リスク
  - 8. 期間を見極める：Phase0〜Phase4で約20週間（5〜8ヶ月）
  - 9. 何を諦めるのか？：品質最優先、時間は多少許容、機能は柔軟に調整
  - 10. 何がどれだけ必要か？：個人開発、Firebase月0〜5,000円、週10〜20時間

### Phase0-4：整合性チェック
- ✅ 3つのドキュメント間の整合性を確認
  - first-idea.mdから「差分検知システム」を削除（スクレイピング廃止に伴い不要）
  - lean-canvas.mdのKPI目標値を現実的な数値に修正
    - Phase3：MAU 50〜100人、DAU 10〜20人（ベータリリース）
    - Phase4：MAU 1,000〜3,000人、DAU 100〜300人（本番リリース）
    - Phase5以降：MAU 10,000人、DAU 1,000人（スケールアップ）
  - 3つのドキュメント間で矛盾なし、Phase0完了

### Phase1-1：ディレクトリ構成の整備
- ✅ Phase0ドキュメントの整理
  - first-idea.md、lean-canvas.md、inception-deck.mdをplanning/ディレクトリに移動
  - planning/README.md作成（Phase0完了の記録）
- ✅ Phase1用ディレクトリの作成
  - specs/：GitHub Spec Kit形式の機能設計書（Phase2 MVP対象機能）
  - adr/：Architecture Decision Records（技術選定の記録）
  - api/：API仕様書（Firestore直接アクセス、Cloud Functions）
  - data-model/：データモデル詳細設計（Firestoreコレクション、セキュリティルール、インデックス）
  - 各ディレクトリにREADME.md作成（役割と作成予定ドキュメントを記載）
- ✅ CLAUDE.md作成
  - AI向けプロジェクト情報（現在のフェーズ、ディレクトリ構造、重要なドキュメント）
  - WRITING_STYLE_GUIDE.mdへのリンク（AI作業時の必須参照）
  - AI向けの注意事項（ドキュメント作成時のルール、作業の優先順位）

---

## 次のアクション

### ✅ Phase0完了
1. ~~**サービス名・プロジェクト名の決定**~~ ✅ 完了（期間限定めし / リミメシ / limimeshi）
2. ~~**GitHubリポジトリの初期設定**~~ ✅ 完了（README.md、.gitignore、CONTRIBUTING.md作成済み）
3. ~~**`first-idea.md` の拡充（Phase0-1）**~~ ✅ 完了（全7セクション + 機能優先順位 + サポート範囲）
4. ~~**記述ルールの策定（WRITING_STYLE_GUIDE.md）**~~ ✅ 完了
5. ~~**`lean-canvas.md` の作成（Phase0-2）**~~ ✅ 完了（9ブロック全て完成）
6. ~~**`inception-deck.md` の作成（Phase0-3）**~~ ✅ 完了（10の質問に全て回答）
7. ~~**整合性チェック（Phase0-4）**~~ ✅ 完了（矛盾を修正、企画ドキュメント完成）

### ✅ Phase1-1完了
8. ~~**ディレクトリ構成の整備**~~ ✅ 完了（planning/, specs/, adr/, api/, data-model/, CLAUDE.md作成）

### 🎯 Phase1：設計・仕様策定（進行中）
**Phase1-2：GitHub Spec Kitで機能設計**
- `specs/menu-list.md`：メニュー一覧機能
- `specs/favorites.md`：お気に入り登録機能
- `specs/admin-panel.md`：管理画面機能

**Phase1-3：ADRで技術選定記録**
- `adr/001-why-firebase.md`：なぜFirebaseを選んだか
- `adr/002-monorepo-vs-multirepo.md`：モノレポ vs マルチリポ
- `adr/003-react-admin.md`：React Adminを選んだ理由
- `adr/004-manual-data-update.md`：データ更新の運用方針

**Phase1-4：データモデル詳細設計**
- `data-model/firestore-collections.md`：Firestoreコレクション設計
- `data-model/security-rules.md`：セキュリティルール設計
- `data-model/indexes.md`：インデックス設計

---

## 更新履歴

- 2025/10/13：初版作成
- 2025/10/13：Phase1-1 の進捗を反映（2/7完了）
- 2025/10/13：Phase1-1 の進捗を反映（3/7完了、スクレイピング方針を追加）
- 2025/10/13：次のアクションに「サービス名・プロジェクト名の決定」を追加
- 2025/10/13：サービス名決定を完了（期間限定めし / リミメシ / limimeshi）、first-idea.mdに記載
- 2025/10/14：GitHubリポジトリ初期設定を開始（README.md、.gitignore、CONTRIBUTING.md作成）
- 2025/10/14：Phase3-1のリポジトリ名をlimimeshi-*に統一
- 2025/10/18：Phase0-1 の進捗を反映（4/7完了、非機能要件を追加）、Phase0の進捗率を50%に更新
- 2025/10/18：Phase0-1 の進捗を反映（5/7完了、開発フェーズ・マイルストーンを追加）、Phase0の進捗率を70%に更新
- 2025/10/18：Phase0-1 の進捗を反映（6/7完了、リスクと対策を追加）、Phase0の進捗率を85%に更新
- 2025/10/18：Phase0-1 の進捗を反映（7/7完了、モニタリング・分析を追加）、Phase0-1を完了
- 2025/10/18：WRITING_STYLE_GUIDE.mdを作成、記述ルールを策定
- 2025/10/18：記述スタイルの統一（句読点、Phase表記、コロン、見出し構造の改善）
- 2025/10/18：Phase表記を統一（「Phase 1:」→「Phase1：」）、Phase0の進捗率を修正（Phase0-1のみ完了で25%）
- 2025/10/24：機能の優先順位を決定（Phase2必須機能とPhase3以降機能を明確化）
- 2025/10/24：OS・ブラウザのサポート範囲を決定（モダンブラウザ最新版、iOS/Android直近3世代）
- 2025/10/24：Phase番号をfirst-idea.mdに合わせて更新（Phase0から開始、Phase5まで定義）
- 2025/10/24：Phase1の設計対象機能をMVP機能のみに限定
- 2025/10/24：データ更新の方針を大幅変更（スクレイピング廃止、完全手動運用＋補助ツール活用）
- 2025/10/25：Phase0-2完了（lean-canvas.md作成、9ブロック全て完成）、Phase0の進捗率を50%に更新
- 2025/10/26：Phase0-3完了（inception-deck.md作成、10の質問に全て回答）、Phase0の進捗率を75%に更新
- 2025/10/26：Phase0-4完了（整合性チェック、矛盾を修正）、Phase0の進捗率を100%に更新、Phase0完了
- 2025/10/26：Phase1-1完了（ディレクトリ構成の整備、planning/, specs/, adr/, api/, data-model/作成、CLAUDE.md作成）、Phase1開始、Phase1の進捗率を25%に更新
- 2025/11/14：Phase1-2の管理画面機能（001-admin-panel）spec.md完成、タグ管理・入力補助機能をOut of Scopeに移動、詳細要件確定（ふりがな必須、X Post URL、ステータス自動判定、チェーン店削除機能なし）
- 2025/11/18：Phase1-3完了（ADR-001〜005作成、Michael Nygard形式採用、タイトル命名規則統一、Constitution/first-idea.mdとの整合性確保）
- 2025/11/19：Phase1-2完了（002-menu-list、003-favoritesのPhase 0, Phase 1, tasks.md生成完了）
- 2025/11/19：リポジトリアーキテクチャの見直し決定（Phase2開始時に各仕様書を実装リポジトリに移行、limimeshi-docsはガバナンス専用に整理）
- 2025/11/22：Phase1-4完了（Firestoreデータベース設計、firestore-best-practices.md、firestore-collections.md、security-rules.md、indexes.md作成）、Phase1完了
