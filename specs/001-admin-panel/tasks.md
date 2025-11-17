# Tasks: 管理画面（Admin Panel）

**入力**: `/specs/001-admin-panel/` の設計ドキュメント
**前提条件**: plan.md, spec.md, research.md, data-model.md, contracts/

**テスト**: 受け入れシナリオ検証のためのE2Eテストを最終フェーズに含む

**構成**: タスクはユーザーストーリーごとにグループ化し、各ストーリーの独立した実装とテストを可能にする

## フォーマット: `[ID] [P?] [Story] 説明`

- **[P]**: 並列実行可能（異なるファイル、依存関係なし）
- **[Story]**: このタスクが属するユーザーストーリー（例: US1, US2, US3）
- 説明に正確なファイルパスを含める

## パス規約

- リポジトリ: `limimeshi-admin/`（別リポジトリ）
- ソース: `limimeshi-admin/src/`
- テスト: `limimeshi-admin/tests/`
- Firestore: `limimeshi-admin/firestore.rules`, `limimeshi-admin/firestore.indexes.json`

**ディレクトリ構造**: React公式推奨のFeature-based構造に準拠（機能ごとにファイルをグループ化）
- 参考: [React公式 - Grouping by features or routes](https://react.dev/learn/thinking-in-react#step-5-add-inverse-data-flow)
- 参考: [React Admin公式デモ](https://github.com/marmelab/react-admin/tree/master/examples/demo/src)

---

## フェーズ1: セットアップ（共通インフラ）

**目的**: プロジェクト初期化と基本構造の構築

- [ ] T001 limimeshi-admin リポジトリを作成し、Gitを初期化
- [ ] T002 limimeshi-admin/package.json で Vite + React + TypeScript プロジェクトを初期化
- [ ] T003 [P] React Admin 4.x 依存関係をインストール（react-admin@^4.16.0, react-admin-firebase@^4.1.0）
- [ ] T004 [P] limimeshi-admin/package.json に Firebase SDK 依存関係をインストール（firebase@^10.13.0）
- [ ] T005 [P] limimeshi-admin/.eslintrc.cjs と limimeshi-admin/.prettierrc で ESLint と Prettier を設定
- [ ] T006 [P] limimeshi-admin/tsconfig.json で TypeScript を設定（target: ES2020, jsx: react-jsx, strict: true）
- [ ] T007 [P] limimeshi-admin/vite.config.ts で Vite を設定（port: 3000, outDir: dist）
- [ ] T008 プロジェクトディレクトリ構造を作成（src/chains/, src/menus/, src/auth/, src/providers/, src/lib/, tests/）
- [ ] T009 [P] Firebase Console で Firebase プロジェクト（limimeshi-dev）をセットアップ
- [ ] T010 [P] limimeshi-admin/.env.example に Firebase 設定変数を含む .env.local テンプレートを作成

---

## フェーズ2: 基盤（全ユーザーストーリーの前提条件）

**目的**: 全てのユーザーストーリー実装前に完了必須のコアインフラ

**⚠️ 重要**: このフェーズ完了まで、ユーザーストーリーの作業を開始できない

- [ ] T011 limimeshi-admin/src/lib/firebase.ts で Firebase を初期化（app, auth, firestore インスタンス）
- [ ] T012 Firebase Console の Authentication タブで Email/Password プロバイダーを有効化
- [ ] T013 Firebase Console の Authentication タブで最初の管理者ユーザーを作成
- [ ] T014 limimeshi-admin/scripts/set-admin-claim.js で管理者ユーザーの Custom Claims 設定スクリプトを作成
- [ ] T015 limimeshi-admin/firestore.rules で Firestore Security Rules をデプロイ（Custom Claims チェックによる管理者専用アクセス）
- [ ] T016 limimeshi-admin/firestore.indexes.json で Firestore 複合インデックスを作成（chainId + saleStartTime DESC）
- [ ] T017 firebase deploy で firestore.rules と firestore.indexes.json を Firebase プロジェクトにデプロイ
- [ ] T018 limimeshi-admin/src/App.tsx で React Admin アプリを初期化（Admin コンポーネント、dataProvider, authProvider プレースホルダー）
- [ ] T019 limimeshi-admin/src/main.tsx でアプリエントリーポイントを作成（React Admin アプリをレンダリング）
- [ ] T020 [P] limimeshi-admin/src/providers/dataProvider.ts で react-admin-firebase データプロバイダーを設定
- [ ] T021 [P] limimeshi-admin/src/providers/authProvider.ts で Firebase 認証プロバイダーを作成（Custom Claims チェック付き）

**チェックポイント**: 基盤準備完了 - ユーザーストーリー実装を並列開始可能

---

## フェーズ3: ユーザーストーリー3 - 管理者認証 (優先度: P1) 🎯 MVP基盤

**目標**: 管理者のみがシステムにアクセスでき、一般ユーザーは管理画面にアクセスできない

**独立テスト**: 管理者でログインできること、一般ユーザーがアクセスできないこと、ログアウトが機能することを確認

**注記**: ユーザーストーリー3（認証）は US1, US2 の前提条件のため、基盤フェーズに含まれる

### ユーザーストーリー3の実装

- [ ] T022 [US3] limimeshi-admin/src/auth/LoginPage.tsx でログインページを実装
- [ ] T023 [US3] limimeshi-admin/src/providers/authProvider.ts の authProvider.checkAuth に Custom Claims 検証を追加
- [ ] T024 [US3] limimeshi-admin/src/providers/authProvider.ts の authProvider.logout でログアウト機能を実装
- [ ] T025 [US3] limimeshi-admin/src/providers/authProvider.ts で誤ったパスワードのエラーハンドリングを追加
- [ ] T026 [US3] limimeshi-admin/src/App.tsx で保護されたルートを設定（未認証時はログイン画面にリダイレクト）
- [ ] T027 [US3] 管理者ユーザーでログインをテスト（手動検証）
- [ ] T028 [US3] 非管理者ユーザーのアクセス拒否をテスト（手動検証）

**チェックポイント**: 認証完了 - チェーン店とメニュー管理を独立して進められる

---

## フェーズ4: ユーザーストーリー1 - チェーン店の管理 (優先度: P1) 🎯 MVP

**目標**: 管理者が新しいチェーン店をシステムに登録し、既存のチェーン店情報を更新できる

**独立テスト**: チェーン店の登録・一覧表示・編集が完全に動作すれば、この機能は独立して価値を提供できる

### ユーザーストーリー1の実装

- [ ] T029 [P] [US1] limimeshi-admin/src/chains/ChainList.tsx で ChainList コンポーネントを作成（List, Datagrid, TextField）
- [ ] T030 [P] [US1] limimeshi-admin/src/chains/ChainEdit.tsx で ChainEdit コンポーネントを作成（Edit, SimpleForm, TextInput）
- [ ] T031 [P] [US1] limimeshi-admin/src/chains/ChainCreate.tsx で ChainCreate コンポーネントを作成（Create, SimpleForm, TextInput）
- [ ] T032 [US1] limimeshi-admin/src/chains/ の ChainEdit と ChainCreate にふりがな検証（ひらがなのみの正規表現）を追加
- [ ] T033 [US1] limimeshi-admin/src/App.tsx で chains リソースを設定（Resource name="chains", list, edit, create）
- [ ] T034 [US1] ChainList にふりがな昇順ソートを追加（sort={{field: 'furigana', order: 'ASC'}}）
- [ ] T035 [US1] ChainList にチェーン名での検索フィルタを追加（filters プロップに TextInput）
- [ ] T036 [US1] 必須項目（name, furigana）でのチェーン作成をテスト - 手動検証
- [ ] T037 [US1] チェーン編集と変更の永続化を検証 - 手動検証
- [ ] T038 [US1] 必須項目未入力時のバリデーションエラーをテスト - 手動検証

**チェックポイント**: チェーン店管理が完了し、独立して機能する

---

## フェーズ5: ユーザーストーリー2 - メニューの管理 (優先度: P1) 🎯 MVP

**目標**: 管理者が期間限定メニューを手動で登録し、販売開始日時と販売終了日時を管理できる

**独立テスト**: メニューの登録・一覧表示・編集・削除が完全に動作し、チェーン店との紐付けが正しく機能する

### ユーザーストーリー2の実装

- [ ] T039 [P] [US2] limimeshi-admin/src/lib/statusUtils.ts でステータス計算ユーティリティを作成（calculateStatus 関数）
- [ ] T040 [P] [US2] limimeshi-admin/tests/unit/statusUtils.test.ts で statusUtils.ts のユニットテストを追加（Vitest）
- [ ] T041 [P] [US2] limimeshi-admin/src/menus/StatusField.tsx で StatusField コンポーネントを作成（計算されたステータスを表示）
- [ ] T042 [P] [US2] limimeshi-admin/src/menus/MenuList.tsx で MenuList コンポーネントを作成（List, Datagrid, ReferenceField, DateField, StatusField）
- [ ] T043 [P] [US2] limimeshi-admin/src/menus/MenuEdit.tsx で MenuEdit コンポーネントを作成（Edit, SimpleForm, ReferenceInput, TextInput, DateTimeInput）
- [ ] T044 [P] [US2] limimeshi-admin/src/menus/MenuCreate.tsx で MenuCreate コンポーネントを作成（Create, SimpleForm, ReferenceInput, TextInput, DateTimeInput）
- [ ] T045 [US2] MenuList のデータ変換にステータス計算を統合（各メニューに status フィールドを追加）
- [ ] T046 [US2] MenuList に saleStartTime 降順ソートを追加（sort={{field: 'saleStartTime', order: 'DESC'}}）
- [ ] T047 [US2] limimeshi-admin/src/App.tsx で menus リソースを設定（Resource name="menus", list, edit, create）
- [ ] T048 [US2] MenuList にメニュー名とチェーン名での検索フィルタを追加（filters プロップ）
- [ ] T049 [US2] MenuList に確認ダイアログ付き削除機能を追加（DeleteButton コンポーネント）
- [ ] T050 [US2] MenuEdit と MenuCreate に saleEndTime > saleStartTime のバリデーションを追加
- [ ] T051 [US2] 必須項目（chainId, name, saleStartTime）でのメニュー作成をテスト - 手動検証
- [ ] T052 [US2] メニュー編集と削除をテスト - 手動検証
- [ ] T053 [US2] 様々な日付シナリオでのステータス自動計算をテスト - 手動検証
- [ ] T054 [US2] saleEndTime <= saleStartTime 時のバリデーションエラーをテスト - 手動検証

**チェックポイント**: メニュー管理が完了し、独立して機能する

---

## フェーズ6: 仕上げ・横断的関心事

**目的**: 複数のユーザーストーリーに影響する改善

- [ ] T055 [P] limimeshi-admin/tests/e2e/auth.spec.ts で管理者ログインフローの E2E テストを追加（Playwright）
- [ ] T056 [P] limimeshi-admin/tests/e2e/chains.spec.ts でチェーン店 CRUD 操作の E2E テストを追加（Playwright）
- [ ] T057 [P] limimeshi-admin/tests/e2e/menus.spec.ts でメニュー CRUD 操作の E2E テストを追加（Playwright）
- [ ] T058 [P] limimeshi-admin/tests/integration/ChainList.test.tsx で ChainList の統合テストを追加（React Testing Library）
- [ ] T059 [P] limimeshi-admin/tests/integration/MenuList.test.tsx で MenuList の統合テストを追加（React Testing Library）
- [ ] T060 パフォーマンス最適化: 必要に応じて MenuList にページネーションを追加（デフォルト perPage: 25）
- [ ] T061 [P] limimeshi-admin/ に README.md を作成（セットアップとデプロイ手順）
- [ ] T062 [P] quickstart.md の手順が end-to-end で動作することを検証（手動検証）
- [ ] T063 コードクリーンアップと未使用インポートの削除
- [ ] T064 最終検証: spec.md の全受け入れシナリオをテスト

---

## 依存関係と実行順序

### フェーズ間の依存関係

- **セットアップ（フェーズ1）**: 依存関係なし - 即座に開始可能
- **基盤（フェーズ2）**: セットアップ完了に依存 - 全ユーザーストーリーをブロック
- **ユーザーストーリー3（フェーズ3）**: 基盤フェーズに依存 - 認証が US1 と US2 をブロック
- **ユーザーストーリー1（フェーズ4）**: 基盤 + US3 完了に依存 - 独立して進められる
- **ユーザーストーリー2（フェーズ5）**: 基盤 + US3 完了に依存 - 独立して進められる（ただしメニュー作成には ReferenceField 用のチェーン店が必要）
- **仕上げ（フェーズ6）**: 全ユーザーストーリー完了に依存

### ユーザーストーリー間の依存関係

- **ユーザーストーリー3（P1）**: 基盤 - US1 と US2 の前に完了必須
- **ユーザーストーリー1（P1）**: US3 完了後に開始可能 - US2 への依存なし
- **ユーザーストーリー2（P1）**: US3 完了後に開始可能 - US1 に弱い依存（メニュー作成にはチェーン店の存在が必要だが、US1 コンポーネントの完成は不要）

### 各ユーザーストーリー内の依存関係

- ファイルを共有しない場合、コンポーネントは並列構築可能
- 実装前にテスト（statusUtils のテストを実装前に作成）
- コンポーネント準備後にリソース設定
- 実装後に手動テスト

### 並列実行の機会

- フェーズ1: T003, T004, T005, T006, T007, T010 を並列実行可能
- フェーズ2: T011-T019 完了後、T020 と T021 を並列実行可能
- フェーズ4（US1）: T029, T030, T031 を並列実行可能
- フェーズ5（US2）: T039, T040, T041, T042, T043, T044 を並列実行可能
- フェーズ6: T055, T056, T057, T058, T059, T061, T062 を並列実行可能

---

## 並列実行の例: ユーザーストーリー1

```bash
# ユーザーストーリー1の全コンポーネントを同時に起動:
Task: "limimeshi-admin/src/chains/ChainList.tsx で ChainList コンポーネントを作成"
Task: "limimeshi-admin/src/chains/ChainEdit.tsx で ChainEdit コンポーネントを作成"
Task: "limimeshi-admin/src/chains/ChainCreate.tsx で ChainCreate コンポーネントを作成"
```

---

## 実装戦略

### MVP優先（クリティカルパス）

1. フェーズ1完了: セットアップ → プロジェクト初期化完了
2. フェーズ2完了: 基盤 → Firebase と React Admin 準備完了
3. フェーズ3完了: ユーザーストーリー3 → 認証動作
4. フェーズ4完了: ユーザーストーリー1 → チェーン店管理動作
5. **一時停止して検証**: US1 + US3 を独立してテスト
6. フェーズ5完了: ユーザーストーリー2 → メニュー管理動作
7. **MVP検証**: 3つのユーザーストーリー全てを一緒にテスト
8. フェーズ6完了: 仕上げ → 本番環境準備完了

### 段階的デリバリー

1. セットアップ + 基盤 + US3 → 管理者がログイン可能
2. US1 追加 → 管理者がチェーン店を管理可能
3. US2 追加 → 管理者がメニューを管理可能（完全な MVP！）
4. 仕上げ追加 → 本番品質

### 並列チーム戦略

複数の開発者がいる場合:

1. チーム全体でセットアップ + 基盤を完了
2. 基盤完了後:
   - 開発者A: ユーザーストーリー3（認証）- 他をブロック
3. US3 完了後:
   - 開発者A: ユーザーストーリー1（チェーン店）
   - 開発者B: ユーザーストーリー2（メニュー）- 並列実行可能だが、テストには少なくとも1つのチェーン店が必要
4. フェーズ6: 仕上げを全員で実施

---

## 注記

- [P] タスク = 異なるファイル、依存関係なし
- [Story] ラベル = 特定のユーザーストーリーへのタスクのマッピング（トレーサビリティ）
- 各ユーザーストーリーは独立して完了・テスト可能であるべき
- US3（認証）は基盤 - US1 と US2 の前に完了必須
- US1 と US2 は弱い結合（US2 はチェーン店の存在が必要だが US1 コンポーネントへの依存はなし）
- 各タスクまたは論理グループ後にコミット
- 任意のチェックポイントで一時停止してストーリーを独立検証
- 手動検証タスクは spec.md の受け入れシナリオを検証すべき
- フェーズ6 の E2E テストは end-to-end のユーザージャーニーを検証
