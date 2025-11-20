# Tasks: メニュー一覧（Menu List）

**Input**: Design documents from `/specs/002-menu-list/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, contracts/firestore-schema.md, quickstart.md

**Tests**: このタスクリストにはテスト作成タスクが含まれています（spec.mdで明示的に要求されているため）

**Organization**: タスクはユーザーストーリーごとにグループ化され、各ストーリーを独立して実装・テスト可能にしています

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 並列実行可能（異なるファイル、依存関係なし）
- **[Story]**: このタスクが属するユーザーストーリー（例: US1, US2）
- 説明には正確なファイルパスを含める

## Path Conventions

- **limimeshi-web repository**: `src/pages/`, `src/components/`, `src/utils/`, `src/types/`, `src/firebase/`, `tests/unit/`, `tests/e2e/`
- パスはplan.mdのProject Structureセクションに基づく

---

## Phase 1: Setup（共有インフラストラクチャ）

**Purpose**: プロジェクトの初期化と基本構造の構築

- [ ] T001 limimeshi-webリポジトリを作成（GitHub、ローカルにクローン）
- [ ] T002 [P] React + TypeScript + Viteプロジェクトを初期化（npm create vite@latest limimeshi-web -- --template react-ts）
- [ ] T003 [P] 必要なパッケージをインストール（Firebase SDK v9+、Material-UI v5、react-twitter-embed、Vitest、React Testing Library、Playwright）
- [ ] T004 [P] プロジェクト構造を作成（src/pages/, src/components/, src/utils/, src/types/, src/firebase/, tests/unit/, tests/e2e/）
- [ ] T005 [P] Firebase設定ファイルを作成 .env.local（VITE_FIREBASE_API_KEY, VITE_FIREBASE_PROJECT_ID, など）
- [ ] T006 [P] Firebase初期化ファイルを作成 src/firebase/config.ts（initializeApp, getFirestore, getAuth）

---

## Phase 2: Foundational（ブロッキング前提条件）

**Purpose**: すべてのユーザーストーリーを実装する前に完了する必要があるコアインフラストラクチャ

**⚠️ CRITICAL**: このフェーズが完了するまで、ユーザーストーリーの作業を開始できません

- [ ] T007 Firestore Security Rulesを確認（/menus, /chainsが公開読み取り可能、書き込み不可）
- [ ] T008 Firestoreインデックスを作成（menus: chainId + saleStartTime desc）
- [ ] T009 TypeScript型定義を作成 src/types/index.ts（Menu, Chain, MenuStatus, MenuWithChain）
- [ ] T010 ステータス判定ロジックのユーティリティ関数を作成 src/utils/menuStatus.ts（calculateMenuStatus: saleStartTime, saleEndTimeからステータスを判定）

**Checkpoint**: 基盤が整い、ユーザーストーリーの並列実装が可能になります

---

## Phase 3: User Story 1 - メニュー一覧の閲覧 (Priority: P1) 🎯 MVP

**Goal**: 一般ユーザーが期間限定メニューを一覧表示し、最新のメニュー情報を素早く確認できる。

**Independent Test**: メニューの一覧表示が正しく動作し、販売開始日時の降順でソートされ、各メニューの詳細情報が表示されれば、独立して価値を提供できる。

### Tests for User Story 1

> **NOTE: これらのテストを最初に書き、実装前に失敗することを確認**

- [ ] T011 [P] [US1] ステータス判定ロジックの単体テストを作成 tests/unit/menuStatus.test.ts（「予定」「発売から◯日経過」「販売終了」の3パターン、販売終了日時未設定時の処理）
- [ ] T012 [P] [US1] メニュー一覧表示のE2Eテストを作成 tests/e2e/menu-list.spec.ts（メニューが販売開始日時降順で表示、1年経過メニューは非表示、空リスト時「データなし」表示）

### Implementation for User Story 1

- [ ] T013 [P] [US1] ステータス判定ロジックを実装 src/utils/menuStatus.ts（calculateMenuStatus関数、現在時刻とsaleStartTime/saleEndTimeを比較）
- [ ] T014 [US1] メニュー一覧画面を作成 src/pages/MenuList.tsx（Firestoreから1年以内のメニューを取得、販売開始日時降順でソート、ローディング状態、エラーハンドリング）
- [ ] T015 [US1] メニューカードコンポーネントを作成 src/components/MenuCard.tsx（チェーン名、メニュー名、販売開始日時、販売終了日時（未設定時は非表示）、ステータス、説明を表示）
- [ ] T016 [US1] X Post埋め込みコンポーネントを作成 src/components/XPostEmbed.tsx（react-twitter-embedを使用、Post ID抽出、エラーハンドリング、Skeleton UI）
- [ ] T017 [US1] メニューカードにX Post埋め込みを統合 src/components/MenuCard.tsx（X Post URLがあれば表示、なければ非表示）
- [ ] T018 [US1] 1年経過フィルタを実装 src/pages/MenuList.tsx（Firestoreクエリで`where('saleStartTime', '>=', oneYearAgo)`）
- [ ] T019 [US1] 空リスト時の表示を実装 src/pages/MenuList.tsx（メニュー0件時に「データなし」と表示）

**Checkpoint**: この時点で、User Story 1は完全に機能し、独立してテスト可能です（メニュー一覧表示機能が動作）

---

## Phase 4: User Story 2 - お気に入りチェーンフィルタ (Priority: P1)

**Goal**: ログインユーザーがお気に入り登録したチェーン店のメニューのみを表示し、関心の高いチェーンの新メニューを素早く確認できる。フィルタ選択は次回訪問時も保持される。

**Independent Test**: お気に入りフィルタが正しく動作し、お気に入り登録済みチェーンのメニューのみが表示され、フィルタ選択が次回訪問時も保持されれば、独立して価値を提供できる。

**Dependencies**: User Story 1（メニュー一覧表示）、003-favorites（お気に入り登録機能）

### Tests for User Story 2

- [ ] T020 [P] [US2] フィルタ選択永続化の単体テストを作成 tests/unit/localStorage.test.ts（localStorageへの保存・読み込み、デフォルト値の処理）
- [ ] T021 [P] [US2] お気に入りフィルタのE2Eテストを作成 tests/e2e/menu-list.spec.ts（フィルタON→お気に入りチェーンのみ表示、フィルタOFF→全メニュー表示、次回訪問時もフィルタ設定保持）

### Implementation for User Story 2

- [ ] T022 [P] [US2] フィルタ選択永続化ユーティリティを作成 src/utils/localStorage.ts（saveFavoritesFilter, loadFavoritesFilter関数）
- [ ] T023 [US2] お気に入りフィルタコンポーネントを作成 src/components/FavoritesFilter.tsx（Material-UI Switch、フィルタON/OFF切り替え、localStorageで永続化）
- [ ] T024 [US2] メニュー一覧画面に認証状態を追加 src/pages/MenuList.tsx（onAuthStateChangedでuserを管理、ログインユーザーのみフィルタ表示）
- [ ] T025 [US2] お気に入りチェーンデータを取得 src/pages/MenuList.tsx（Firestoreから/users/{userId}/favoritesを取得、chainIdリストを作成）
- [ ] T026 [US2] お気に入りフィルタロジックを実装 src/pages/MenuList.tsx（フィルタON時は`where('chainId', 'in', favoriteChainIds)`でクエリ、10件制限の考慮）
- [ ] T027 [US2] フィルタ選択永続化を統合 src/pages/MenuList.tsx（初回表示時にlocalStorageから設定を読み込み、フィルタ変更時に保存）
- [ ] T028 [US2] お気に入りチェーン0件時の処理 src/pages/MenuList.tsx（フィルタON時にお気に入り0件の場合は「データなし」と表示）

**Checkpoint**: この時点で、User Stories 1 AND 2は両方とも独立して動作します（メニュー一覧表示＋お気に入りフィルタ）

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: 複数のユーザーストーリーに影響する改善

- [ ] T029 [P] Firestore Security Rulesのデプロイ確認（Firebase Consoleで設定、またはfirebase deployコマンド）
- [ ] T030 [P] Firestoreインデックスのデプロイ確認（Firebase Consoleで設定、またはfirebaserc + firestore.indexesファイル）
- [ ] T031 [P] パフォーマンス検証（3秒以内の初期表示、モバイル4G環境、X Post埋め込み含む）
- [ ] T032 [P] エラーハンドリングの網羅性確認（Firestoreエラー、X Post埋め込みエラー、その他のエラーケース）
- [ ] T033 [P] アクセシビリティ改善（aria-labelの追加、スクリーンリーダー対応、キーボード操作）
- [ ] T034 [P] レスポンシブデザインの検証（iOS Safari, Android Chrome、モバイル4G環境）
- [ ] T035 [P] テストカバレッジの確認（70%以上のターゲット達成確認）
- [ ] T036 [P] 負荷テスト（100ユーザー想定、Firestoreクエリの同時実行テスト）
- [ ] T037 Firebase Hosting設定を確認 firebase.json（hosting設定が正しいことを確認）

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

### 総タスク数: 37タスク

### ユーザーストーリーごとのタスク数:
- **User Story 1（メニュー一覧の閲覧）**: 11タスク（テスト2 + 実装9）
- **User Story 2（お気に入りチェーンフィルタ）**: 9タスク（テスト2 + 実装7）

### 各ストーリーの独立テスト基準:
- **User Story 1**: メニューの一覧表示が正しく動作し、販売開始日時の降順でソートされ、各メニューの詳細情報が表示される
- **User Story 2**: お気に入りフィルタが正しく動作し、お気に入り登録済みチェーンのメニューのみが表示され、フィルタ選択が次回訪問時も保持される

### 推奨MVPスコープ:
- **Phase 1 + Phase 2 + Phase 3（User Story 1のみ）**: メニュー一覧表示機能
- User Story 2はお気に入り機能との連携として有用だが、User Story 1完了後に追加可能
