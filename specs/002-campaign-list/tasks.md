# Tasks: キャンペーン一覧（Campaign List）

**Input**: Design documents from `/specs/002-campaign-list/`  
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, contracts/firestore-schema.md, quickstart.md  

**Tests**: このタスクリストにはテスト作成タスクが含まれています（spec.mdで明示的に要求されているため）

**Organization**: タスクはユーザーストーリーごとにグループ化され、各ストーリーを独立して実装・テスト可能にしています

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 並列実行可能（異なるファイル、依存関係なし）
- **[Story]**: このタスクが属するユーザーストーリー（例: US1, US2）
- 説明には正確なファイルパスを含める

## Path Conventions

- **limimeshi-android repository**: `app/src/main/java/com/limimeshi/android/`, `app/src/test/`, `app/src/androidTest/`
- パスはplan.mdのProject Structureセクションに基づく

---

## Phase 1: Setup（共有インフラストラクチャ）

**Purpose**: プロジェクトの初期化と基本構造の構築

- [ ] T001 limimeshi-androidリポジトリを作成（GitHub、ローカルにクローン）
- [ ] T002 [P] Android Studio でKotlin + Jetpack Composeプロジェクトを初期化（Empty Activity with Compose）
- [ ] T003 [P] 必要なパッケージをインストール（Firebase SDK、Hilt、DataStore、JUnit、MockK、Turbine、Compose Testing）
- [ ] T004 [P] プロジェクト構造を作成（ui/, data/repository/, data/model/, util/, di/）
- [ ] T005 [P] Firebase設定ファイルを配置 google-services.json（Firebase Consoleからダウンロード）
- [ ] T006 [P] Hilt DIセットアップ di/AppModule.kt、LimimeshiApp.kt（@HiltAndroidApp）

---

## Phase 2: Foundational（ブロッキング前提条件）

**Purpose**: すべてのユーザーストーリーを実装する前に完了する必要があるコアインフラストラクチャ

**⚠️ CRITICAL**: このフェーズが完了するまで、ユーザーストーリーの作業を開始できません

- [ ] T007 Firestore Security Rulesを確認（/campaigns, /chainsが公開読み取り可能、書き込み不可）
- [ ] T008 Firestoreインデックスを作成（campaigns: chainId + saleStartTime desc）
- [ ] T009 Kotlinデータクラス定義を作成 data/model/Campaign.kt、Chain.kt、CampaignStatus.kt
- [ ] T010 ステータス判定ロジックのユーティリティ関数を作成 util/CampaignStatusUtil.kt（getCampaignStatus: saleStartTime, saleEndTimeからステータスを判定）

**Checkpoint**: 基盤が整い、ユーザーストーリーの並列実装が可能になります

---

## Phase 3: User Story 1 - キャンペーン一覧の閲覧 (Priority: P1) 🎯 MVP

**Goal**: 一般ユーザーが期間限定キャンペーンを一覧表示し、最新のキャンペーン情報を素早く確認できる。

**Independent Test**: キャンペーンの一覧表示が正しく動作し、販売開始日時の降順でソートされ、各キャンペーンの詳細情報が表示されれば、独立して価値を提供できる。

### Tests for User Story 1

> **NOTE: これらのテストを最初に書き、実装前に失敗することを確認**

- [ ] T011 [P] [US1] ステータス判定ロジックの単体テストを作成 app/src/test/java/.../util/CampaignStatusUtilTest.kt（「予定」「開始から◯日経過」「終了」の3パターン、販売終了日時未設定時の処理）
- [ ] T012 [P] [US1] キャンペーン一覧表示のUIテストを作成 app/src/androidTest/java/.../ui/campaign/CampaignListScreenTest.kt（キャンペーンが販売開始日時降順で表示、1年経過キャンペーンは非表示、空リスト時「データなし」表示）

### Implementation for User Story 1

- [ ] T013 [P] [US1] ステータス判定ロジックを実装 util/CampaignStatusUtil.kt（getCampaignStatus関数、現在時刻とsaleStartTime/saleEndTimeを比較）
- [ ] T014 [US1] CampaignRepositoryを作成 data/repository/CampaignRepository.kt（Firestoreから1年以内のキャンペーンを取得、販売開始日時降順でソート）
- [ ] T015 [US1] CampaignListViewModelを作成 ui/campaign/CampaignListViewModel.kt（UIステート管理、キャンペーン取得、エラーハンドリング）
- [ ] T016 [US1] キャンペーン一覧画面を作成 ui/campaign/CampaignListScreen.kt（LazyColumn、ローディング状態、エラー状態、空リスト状態）
- [ ] T017 [US1] キャンペーンカードコンポーネントを作成 ui/campaign/CampaignCard.kt（チェーン名、キャンペーン名、販売開始日時、販売終了日時（未設定時は非表示）、ステータス、説明を表示）
- [ ] T018 [US1] X Post埋め込みコンポーネントを作成 ui/components/XPostEmbed.kt（WebViewでTwitter Widgets使用、Post ID抽出、エラーハンドリング）
- [ ] T019 [US1] キャンペーンカードにX Post埋め込みを統合 ui/campaign/CampaignCard.kt（X Post URLがあれば表示、なければ非表示）
- [ ] T020 [US1] 1年経過フィルタを実装 data/repository/CampaignRepository.kt（Firestoreクエリで`whereGreaterThanOrEqualTo("saleStartTime", oneYearAgo)`）
- [ ] T021 [US1] 空リスト時の表示を実装 ui/campaign/CampaignListScreen.kt（キャンペーン0件時に「データなし」と表示）

**Checkpoint**: この時点で、User Story 1は完全に機能し、独立してテスト可能です（キャンペーン一覧表示機能が動作）

---

## Phase 4: User Story 2 - お気に入りチェーンフィルタ (Priority: P1)

**Goal**: ログインユーザーがお気に入り登録したチェーン店のキャンペーンのみを表示し、関心の高いチェーンの新キャンペーンを素早く確認できる。フィルタ選択は次回訪問時も保持される。

**Independent Test**: お気に入りフィルタが正しく動作し、お気に入り登録済みチェーンのキャンペーンのみが表示され、フィルタ選択が次回訪問時も保持されれば、独立して価値を提供できる。

**Dependencies**: User Story 1（キャンペーン一覧表示）、003-favorites（お気に入り登録機能）

### Tests for User Story 2

- [ ] T022 [P] [US2] フィルタ選択永続化の単体テストを作成 app/src/test/java/.../data/repository/PreferencesRepositoryTest.kt（DataStoreへの保存・読み込み、デフォルト値の処理）
- [ ] T023 [P] [US2] お気に入りフィルタのUIテストを作成 app/src/androidTest/java/.../ui/campaign/CampaignListScreenTest.kt（フィルタON→お気に入りチェーンのみ表示、フィルタOFF→全キャンペーン表示、次回起動時もフィルタ設定保持）

### Implementation for User Story 2

- [ ] T024 [P] [US2] PreferencesRepositoryを作成 data/repository/PreferencesRepository.kt（DataStore Preferences、showFavoritesOnly保存・読み込み）
- [ ] T025 [US2] お気に入りフィルタコンポーネントを作成 ui/components/FavoritesFilter.kt（Material 3 Switch、フィルタON/OFF切り替え）
- [ ] T026 [US2] CampaignListViewModelに認証状態を追加 ui/campaign/CampaignListViewModel.kt（Firebase Auth AuthStateListener、ログインユーザーのみフィルタ表示）
- [ ] T027 [US2] お気に入りチェーンデータを取得 data/repository/FavoritesRepository.kt（Firestoreから/users/{userId}/favoritesを取得、chainIdリストを作成）
- [ ] T028 [US2] お気に入りフィルタロジックを実装 data/repository/CampaignRepository.kt（フィルタON時は`whereIn("chainId", favoriteChainIds)`でクエリ、10件制限の考慮）
- [ ] T029 [US2] フィルタ選択永続化を統合 ui/campaign/CampaignListViewModel.kt（初回起動時にDataStoreから設定を読み込み、フィルタ変更時に保存）
- [ ] T030 [US2] お気に入りチェーン0件時の処理 ui/campaign/CampaignListScreen.kt（フィルタON時にお気に入り0件の場合は「データなし」と表示）

**Checkpoint**: この時点で、User Stories 1 AND 2は両方とも独立して動作します（キャンペーン一覧表示＋お気に入りフィルタ）

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: 複数のユーザーストーリーに影響する改善

- [ ] T031 [P] Firestore Security Rulesのデプロイ確認（Firebase Consoleで設定、またはfirebase deployコマンド）
- [ ] T032 [P] Firestoreインデックスのデプロイ確認（Firebase Consoleで設定）
- [ ] T033 [P] パフォーマンス検証（3秒以内の初期表示、モバイル4G環境、X Post埋め込み部分を除く）
- [ ] T034 [P] エラーハンドリングの網羅性確認（Firestoreエラー、X Post埋め込みエラー、ネットワークエラー）
- [ ] T035 [P] アクセシビリティ改善（contentDescription追加、TalkBack対応、コントラスト比確認）
- [ ] T036 [P] Material 3テーマ設定確認 ui/theme/Theme.kt（ダイナミックカラー、ダークモード対応）
- [ ] T037 [P] テストカバレッジの確認（70%以上のターゲット達成確認）
- [ ] T038 [P] ProGuard/R8設定確認（リリースビルドで動作確認）
- [ ] T039 Google Play Console設定を確認（内部テストトラック、アプリ署名）

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 依存関係なし - 即座に開始可能
- **Foundational (Phase 2)**: Setupの完了に依存 - すべてのユーザーストーリーをブロック
- **User Stories (Phase 3+)**: すべてFoundationalフェーズの完了に依存
  - User Story 1: Foundational完了後に開始可能 - 他のストーリーへの依存なし
  - User Story 2: User Story 1完了後に開始可能 - お気に入り機能（003-favorites）に依存
- **Polish (Final Phase)**: すべての希望するユーザーストーリーの完了に依存

### User Story Dependencies

- **User Story 1 (P1)**: Foundational (Phase 2)完了後に開始可能 - 他のストーリーへの依存なし
- **User Story 2 (P1)**: User Story 1完了後に開始可能 - 003-favorites（お気に入り登録機能）に依存

---

## 成果物の確認

### 総タスク数: 39タスク

### ユーザーストーリーごとのタスク数:
- **User Story 1（キャンペーン一覧の閲覧）**: 11タスク（テスト2 + 実装9）
- **User Story 2（お気に入りチェーンフィルタ）**: 9タスク（テスト2 + 実装7）

### 各ストーリーの独立テスト基準:
- **User Story 1**: キャンペーンの一覧表示が正しく動作し、販売開始日時の降順でソートされ、各キャンペーンの詳細情報が表示される
- **User Story 2**: お気に入りフィルタが正しく動作し、お気に入り登録済みチェーンのキャンペーンのみが表示され、フィルタ選択が次回訪問時も保持される

### 推奨MVPスコープ:
- **Phase 1 + Phase 2 + Phase 3（User Story 1のみ）**: キャンペーン一覧表示機能
- User Story 2はお気に入り機能との連携として有用だが、User Story 1完了後に追加可能
