# Implementation Plan: メニュー一覧（Menu List）

**Branch**: `002-menu-list` | **Date**: 2025-11-19 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/002-menu-list/spec.md`  

## Summary

一般ユーザー向けのメニュー一覧画面。期間限定メニューを一覧表示し、ログインユーザーはお気に入り登録したチェーンのみ表示するフィルタを利用可能。各メニューにはチェーン名、メニュー名、販売開始日時、販売終了日時（未設定時は非表示）、ステータス（予定/発売から◯日経過/販売終了）、説明を表示。X Post URLがあれば埋め込み表示（商品画像代わり）。販売開始日時の降順（新しい順）でソート。1年経過したメニューは非表示。ログインユーザーのフィルタ選択は次回訪問時も保持。Phase2 MVPで実装。

**技術アプローチ**:
- React 18 + TypeScript + Vite のSPAアプリケーション
- Firebase JS SDK v9+ のFirestoreでデータ読み取り（管理画面が登録済み）
- Material-UI（MUI）のコンポーネントでUI実装
- react-twitter-embed でX Post埋め込み表示
- localStorageでフィルタ選択を永続化

## Technical Context

**Language/Version**: TypeScript 5.x, React 18
**Primary Dependencies**: Firebase JS SDK v9+, Material-UI v5, react-twitter-embed, Vite 5  
**Storage**: Firestore（読み取り専用）  
**Testing**: Vitest, React Testing Library, Playwright  
**Target Platform**: Web（SPA）、モバイル対応（iOS Safari, Android Chrome）  
**Project Type**: Web（limimeshi-web リポジトリ）  
**Performance Goals**: 初期表示3秒以内（モバイル4G環境、X Post埋め込み含む）  
**Constraints**: Firestore読み取り150件以内（1年以内のメニュー、1年経過したメニューは自動非表示）  
**Scale/Scope**: Phase2の160ユーザー、16チェーン、平均メニュー数は変動  
  
## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ✅ I. Spec-Driven（仕様駆動開発）
- **Status**: PASS
- **Evidence**: spec.md、plan.md、tasks.md 作成済み

### ✅ II. Test-First（テスト駆動）
- **Status**: PASS
- **Plan**: ステータス判定ロジック、フィルタロジック、1年経過フィルタロジックの単体テストを先に作成、E2Eテストも先に作成

### ✅ III. Simplicity（シンプルさ優先）
- **Status**: PASS
- **Evidence**: useState + useEffect のみ使用（Redux/Zustandなどのグローバル状態管理ライブラリ不使用、YAGNI原則）

### ✅ IV. Firebase-First
- **Status**: PASS
- **Evidence**: Firebase JS SDK v9+、Firestore、Firebase Authentication 使用

### ✅ V. Legal Risk Zero（法的リスクゼロ）
- **Status**: PASS
- **Evidence**: 読み取り専用機能、管理画面が手動登録したデータを表示するのみ（スクレイピング不使用）

### ✅ VI. Mobile & Performance First
- **Status**: PASS
- **Evidence**: Material-UI（モバイル対応）、初期表示3秒以内、Firestore読み取り最適化

### ✅ VII. Cost Awareness（コスト意識）
- **Status**: PASS
- **Evidence**: Firestore読み取り最適化（インデックス最適化）、Phase2では144回読み取り（ユーザー数想定）

### ✅ VIII. Security & Privacy First
- **Status**: PASS
- **Evidence**: Firestore Security Rules（公開読み取り可能だが書き込み不可）、個人情報の扱いなし（お気に入りデータは別機能）

### ✅ IX. Observability（可観測性）
- **Status**: PASS (Phase2向け)
- **Plan**: Firebase Console、Google Analytics 4 で画面表示回数を監視、Phase2では最小限

**Overall**: ✅ ALL GATES PASSED

## Project Structure

### Documentation (this feature)

```text
specs/002-menu-list/
├── spec.md              # 機能仕様書（既存）
├── plan.md              # 実装計画（/speckit-plan コマンドで生成）
├── research.md          # Phase 0 研究（既存）
├── quickstart.md        # Phase 1 研究（既存）
├── contracts/           # Phase 1 研究（既存）
│   └── firestore-schema.md
└── tasks.md             # Phase 2 研究（/speckit-tasks コマンドで生成）
```

### Source Code (limimeshi-web repository)

```text
limimeshi-web/
├── src/
│   ├── pages/
│   │   └── MenuList.tsx            # メニュー一覧画面
│   ├── components/
│   │   ├── MenuCard.tsx            # メニューカードコンポーネント
│   │   ├── FavoritesFilter.tsx    # お気に入りフィルタコンポーネント
│   │   └── XPostEmbed.tsx          # X Post埋め込みコンポーネント
│   ├── utils/
│   │   ├── menuStatus.ts           # ステータス判定ロジック
│   │   └── localStorage.ts         # フィルタ選択の永続化
│   ├── types/
│   │   └── index.ts                # 型定義（Menu, Chain, MenuStatusなど）
│   ├── firebase/
│   │   └── config.ts               # Firebase設定
│   └── App.tsx                     # ルーティング
├── tests/
│   ├── unit/
│   │   └── menuStatus.test.ts      # ステータス判定の単体テスト
│   └── e2e/
│       └── menu-list.spec.ts       # E2Eテスト
├── package.json
├── vite.config.ts
├── tsconfig.json
└── .env.local                      # Firebase設定（環境変数）
```

**Structure Decision**: Feature-basedではなくReact標準構成、`pages/`に画面コンポーネント、`components/`に再利用可能なコンポーネント、`utils/`にビジネスロジック、`types/`に型定義（001-admin-panelとは異なる構成）

**Note**: React標準構成（Feature-basedではない）、React Adminは管理画面にのみ使用

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

N/A - この機能のConstitution違反なし

## Implementation Phases

### Phase 0: Research（研究）✅

**成果物**: `research.md`

**調査項目**:
1. React + TypeScript（フロントエンド基盤）
2. Firestore SDK（データ読み取り）
3. Firebase Authentication（ログイン状態の管理）
4. X（Twitter）Post埋め込み
5. フィルタ選択の永続化（localStorage）
6. Firestore インデックス
7. ステータス判定ロジック
8. UI/UX設計（Material-UI）
9. 状態管理（useState + useEffect）
10. テスト戦略（Vitest + Playwright）

**この機能の特殊性はConstitutionに準拠しており、追記不要**

### Phase 1: Design & Contracts（設計）✅

**成果物**:
- ✅ `contracts/firestore-schema.md`: Firestoreスキーマ（読み取り専用）
- ✅ `quickstart.md`: 開発環境構築、実装例、テスト例

**設計内容**:
- Firestoreコレクション（/menus, /chains, /users/{userId}/favorites）
- 画面設計（メニュー一覧、お気に入りフィルタ）
- ステータスロジック設計（saleStartTime、saleEndTime）
- ソートロジック（chainId + saleStartTime）
- TypeScript型定義（Menu, Chain, MenuStatus, MenuWithChain）
- Security Rules（読み取り専用）
- パフォーマンス最適化（Firestore読み取り最適化、インデックス最適化）

**001-admin-panelとの整合性**: /menus, /chainsのスキーマは001-admin-panelと完全一致

### Phase 2: Implementation（tasks.md生成）

**Phase 2は `/speckit-tasks` コマンドで tasks.md 生成を実行**

**実装フロー概要**（詳細はtasks.mdに記載）:
1. **環境構築**
   - limimeshi-webリポジトリ初期化
   - Firebase SDK、Material-UI、react-twitter-embedなどをインストール
   - Firebase設定（.env.local）
   - Firestoreインデックス初期化

2. **実装順序（TDD）**
   - ステータス判定ロジック（menuStatus.ts）の単体テスト作成
   - ステータス判定ロジック実装
   - メニュー一覧画面（MenuList.tsx）実装
   - お気に入りフィルタ（FavoritesFilter.tsx）実装
   - X Post埋め込み（XPostEmbed.tsx）実装
   - フィルタ選択の永続化（localStorage.ts）実装

3. **E2Eテスト**
   - メニュー一覧表示のE2Eテスト
   - お気に入りフィルタのE2Eテスト
   - フィルタ選択永続化のE2Eテスト

4. **パフォーマンス最適化**
   - Firestore読み取り最適化（インデックス最適化）
   - 初期表示の高速化（遅延読み込み最適化）
   - モバイル4G環境で3秒以内の表示確認

5. **デプロイ**
   - Firebase Hosting設定（limimeshi-web.web.app）
   - 開発環境デプロイ
   - 動作確認

## Dependencies

### 前提条件
- **001-admin-panel**: チェーン店とメニューが管理画面で登録済み（前提条件、読み取り専用）
- **003-favorites**: お気に入りデータが必要（前提条件、お気に入りフィルタ機能）

### 提供するデータ
- **003-favorites**: お気に入り機能がメニュー一覧画面（002）を表示

### 並行作業の可能性
- 001-admin-panelが完了すれば並行作業可能、ただしFirestoreスキーマの整合性に注意
- 003-favoritesは並行作業不可（お気に入りフィルタは003完了後に追加）

## Risks & Mitigation

### Risk 1: X Post埋め込みが表示できない
- **Impact**: 商品画像が表示できず、視覚的魅力が低下
- **Probability**: 中（PostのIDやAPIの問題）
- **Mitigation**:
  - Post IDの正規表現バリデーションをテスト
  - PostのIDが無効な場合はフォールバック表示（説明文のみ表示）
  - react-twitter-embedのエラーハンドリング

### Risk 2: Firestore読み取り回数が想定超過
- **Impact**: コスト増加
- **Probability**: 低（Phase2では144回程度、150回以内）
- **Mitigation**:
  - Firestore読み取り最適化（インデックス最適化）
  - 初期表示の高速化（遅延読み込み最適化）
  - Firebase Consoleで監視

### Risk 3: 初期表示が遅い
- **Impact**: ユーザー体験悪化
- **Probability**: 中（X Post埋め込みの影響）
- **Mitigation**:
  - X Post埋め込みは非同期ロード
  - メニュー一覧は先に表示、X Postは後から表示
  - Skeleton UIで読み込み中を表示

### Risk 4: お気に入りチェーンが10件超過
- **Impact**: Firestoreの`where('chainId', 'in', ...)`制約（最大10件）
- **Probability**: 低（Phase2では一般ユーザーがそこまで登録しない）
- **Mitigation**:
  - Phase2ではお気に入りチェーン10件までに制限
  - Phase3で複数回クエリに分割する実装に変更

## Success Metrics

### Phase2完了時の基準
- ✅ メニュー一覧の初期表示が3秒以内（モバイル4G環境、X Post埋め込み含む）
- ✅ メニューがスムーズにスクロールできる（10件以上ある場合）
- ✅ お気に入りフィルタの切り替えが1秒以内に反映される
- ✅ ステータスが0件の場合は正しい表示がされる（空リストではなく「データなし」と表示）
- ✅ フィルタ選択が次回訪問時も保持される確率が100%である（localStorageで永続化）
- ✅ 単体テストカバレッジ70%以上
- ✅ E2Eテストが全て合格

### Phase3以降の目標
- 同時アクセス500人が可能な負荷テスト合格
- 初期表示2秒以内（フィルタ選択あり）
- リアルタイム更新機能
- パフォーマンス最適化（遅延読み込み）

## Notes

- **読み取り専用**: 002-menu-listはメニューの読み取り専用機能（書き込みは001-admin-panelと003-favoritesが担当）
- **001との整合性**: /menus、/chainsのスキーマは001-admin-panelと完全一致
- **003との依存関係**: お気に入りフィルタは003-favorites機能に依存（003が未完了の場合はフィルタ選択不可）
- **X Post埋め込み**: Phase2で商品画像の代わりに実装（代替画像なし）
- **1年経過フィルタ**: 販売開始日時から1年経過したメニューは自動的に非表示（1年経過したメニューは削除しない、非表示のみ）
