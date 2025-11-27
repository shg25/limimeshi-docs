# Implementation Plan: キャンペーン一覧（Campaign List）

**Branch**: `002-campaign-list` | **Date**: 2025-11-28 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/002-campaign-list/spec.md`  

## Summary

一般ユーザー向けのキャンペーン一覧画面（Androidアプリ）。期間限定キャンペーンを一覧表示し、ログインユーザーはお気に入り登録したチェーンのみ表示するフィルタを利用可能。各キャンペーンにはチェーン名、キャンペーン名、販売開始日時、販売終了日時（未設定時は非表示）、ステータス（予定/開始から◯日経過/終了）、説明を表示。X Post URLがあれば埋め込み表示（商品画像代わり）。販売開始日時の降順（新しい順）でソート。1年経過したキャンペーンは非表示。ログインユーザーのフィルタ選択は次回訪問時も保持。Phase2 MVPで実装。

**技術アプローチ**:
- Kotlin + Jetpack Compose + Material 3 のネイティブAndroidアプリ
- Firebase Android SDK（Firestore、Authentication）でデータ読み取り（管理画面が登録済み）
- MVVM + Clean Architecture（簡略版）
- WebView でX Post埋め込み表示
- DataStore Preferences でフィルタ選択を永続化

## Technical Context

**Language/Version**: Kotlin 1.9+, Android SDK 34 (Android 14)  
**Primary Dependencies**: Jetpack Compose 1.5+, Material 3, Firebase Android SDK, DataStore, Hilt  
**Storage**: Firestore（読み取り専用）、DataStore Preferences（フィルタ設定）  
**Testing**: JUnit 5, MockK, Turbine, Compose Testing, Robolectric  
**Target Platform**: Android 8.0+ (API 26+)  
**Project Type**: Mobile（limimeshi-android リポジトリ）  
**Performance Goals**: 初期表示3秒以内（モバイル4G環境、X Post埋め込み部分を除く）  
**Constraints**: Firestore読み取り150件以内（1年以内のキャンペーン、1年経過したキャンペーンは自動非表示）  
**Scale/Scope**: Phase2の160ユーザー、16チェーン、平均キャンペーン数は変動  

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### ✅ I. Spec-Driven（仕様駆動開発）
- **Status**: PASS
- **Evidence**: spec.md、plan.md、tasks.md 作成済み

### ✅ II. Test-First（テスト駆動）
- **Status**: PASS
- **Plan**: ステータス判定ロジック、フィルタロジック、1年経過フィルタロジックの単体テストを先に作成、UIテストも先に作成

### ✅ III. Simplicity（シンプルさ優先）
- **Status**: PASS
- **Evidence**: MVVM + StateFlow のみ使用（複雑な状態管理ライブラリ不使用、YAGNI原則）

### ✅ IV. Firebase-First
- **Status**: PASS
- **Evidence**: Firebase Android SDK、Firestore、Firebase Authentication 使用

### ✅ V. Legal Risk Zero（法的リスクゼロ）
- **Status**: PASS
- **Evidence**: 読み取り専用機能、管理画面が手動登録したデータを表示するのみ（スクレイピング不使用）

### ✅ VI. Mobile & Performance First
- **Status**: PASS
- **Evidence**: ネイティブAndroidアプリ、Jetpack Compose、初期表示3秒以内、Firestoreキャッシュ活用

### ✅ VII. Cost Awareness（コスト意識）
- **Status**: PASS
- **Evidence**: Firestore読み取り最適化（インデックス最適化）、Phase2では144回読み取り（ユーザー数想定）

### ✅ VIII. Security & Privacy First
- **Status**: PASS
- **Evidence**: Firestore Security Rules（公開読み取り可能だが書き込み不可）、個人情報の扱いなし（お気に入りデータは別機能）

### ✅ IX. Observability（可観測性）
- **Status**: PASS (Phase2向け)
- **Plan**: Firebase Console、Firebase Crashlytics で画面表示回数・クラッシュを監視、Phase2では最小限

**Overall**: ✅ ALL GATES PASSED

## Project Structure

### Documentation (this feature)

```text
specs/002-campaign-list/
├── spec.md              # 機能仕様書（既存）
├── plan.md              # 実装計画（/speckit-plan コマンドで生成）
├── research.md          # Phase 0 研究（既存）
├── quickstart.md        # Phase 1 研究（既存）
├── contracts/           # Phase 1 研究（既存）
│   └── firestore-schema.md
└── tasks.md             # Phase 2 研究（/speckit-tasks コマンドで生成）
```

### Source Code (limimeshi-android repository)

```text
limimeshi-android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/limimeshi/android/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── campaign/
│   │   │   │   │   │   ├── CampaignListScreen.kt      # キャンペーン一覧画面
│   │   │   │   │   │   ├── CampaignListViewModel.kt   # ViewModel
│   │   │   │   │   │   └── CampaignCard.kt            # キャンペーンカードコンポーネント
│   │   │   │   │   ├── components/
│   │   │   │   │   │   ├── FavoritesFilter.kt         # お気に入りフィルタコンポーネント
│   │   │   │   │   │   └── XPostEmbed.kt              # X Post埋め込みコンポーネント
│   │   │   │   │   └── theme/
│   │   │   │   │       └── Theme.kt                   # Material 3テーマ
│   │   │   │   ├── data/
│   │   │   │   │   ├── repository/
│   │   │   │   │   │   ├── CampaignRepository.kt      # キャンペーンデータ取得
│   │   │   │   │   │   └── PreferencesRepository.kt   # フィルタ設定永続化
│   │   │   │   │   └── model/
│   │   │   │   │       ├── Campaign.kt                # キャンペーンモデル
│   │   │   │   │       ├── Chain.kt                   # チェーンモデル
│   │   │   │   │       └── CampaignStatus.kt          # ステータス（Sealed Class）
│   │   │   │   ├── util/
│   │   │   │   │   └── CampaignStatusUtil.kt          # ステータス判定ロジック
│   │   │   │   ├── di/
│   │   │   │   │   └── AppModule.kt                   # Hilt DI設定
│   │   │   │   └── LimimeshiApp.kt                    # Applicationクラス
│   │   │   └── res/
│   │   │       └── values/
│   │   │           └── strings.xml
│   │   ├── test/
│   │   │   └── java/com/limimeshi/android/
│   │   │       ├── util/
│   │   │       │   └── CampaignStatusUtilTest.kt      # ステータス判定の単体テスト
│   │   │       └── ui/campaign/
│   │   │           └── CampaignListViewModelTest.kt   # ViewModelの単体テスト
│   │   └── androidTest/
│   │       └── java/com/limimeshi/android/
│   │           └── ui/campaign/
│   │               └── CampaignListScreenTest.kt      # UIテスト
│   ├── build.gradle.kts
│   └── google-services.json                           # Firebase設定（.gitignore）
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

**Structure Decision**: Android公式推奨のMVVM構成、`ui/`に画面・ViewModel・コンポーネント、`data/`にRepository・Model、`util/`にユーティリティ

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

N/A - この機能のConstitution違反なし

## Implementation Phases

### Phase 0: Research（研究）✅

**成果物**: `research.md`

**調査項目**:
1. Kotlin + Jetpack Compose（Androidフロントエンド基盤）
2. Firebase Android SDK（データ読み取り）
3. Firebase Authentication（ログイン状態の管理）
4. X（Twitter）Post埋め込み（WebView）
5. フィルタ選択の永続化（DataStore Preferences）
6. Firestore インデックス
7. ステータス判定ロジック
8. アーキテクチャパターン（MVVM + Clean Architecture）
9. 状態管理（StateFlow + Compose State）
10. テスト戦略（JUnit 5 + MockK + Compose Testing）

**この機能の特殊性はConstitutionに準拠しており、追記不要**

### Phase 1: Design & Contracts（設計）✅

**成果物**:
- ✅ `contracts/firestore-schema.md`: Firestoreスキーマ（読み取り専用）
- ✅ `quickstart.md`: 開発環境構築、実装例、テスト例

**設計内容**:
- Firestoreコレクション（/campaigns, /chains, /users/{userId}/favorites）
- 画面設計（キャンペーン一覧、お気に入りフィルタ）
- ステータスロジック設計（saleStartTime、saleEndTime）
- ソートロジック（chainId + saleStartTime）
- Kotlinデータクラス定義（Campaign, Chain, CampaignStatus, CampaignWithChain）
- Security Rules（読み取り専用）
- パフォーマンス最適化（Firestore読み取り最適化、インデックス最適化）

**001-admin-panelとの整合性**: /campaigns, /chainsのスキーマは001-admin-panelと完全一致

### Phase 2: Implementation（tasks.md生成）

**Phase 2は `/speckit-tasks` コマンドで tasks.md 生成を実行**

**実装フロー概要**（詳細はtasks.mdに記載）:
1. **環境構築**
   - limimeshi-androidリポジトリ初期化
   - Android Studio プロジェクト作成（Kotlin + Compose）
   - Firebase SDK、Hilt、DataStoreなどをインストール
   - Firebase設定（google-services.json）
   - Firestoreインデックス初期化

2. **実装順序（TDD）**
   - ステータス判定ロジック（CampaignStatusUtil.kt）の単体テスト作成
   - ステータス判定ロジック実装
   - キャンペーン一覧画面（CampaignListScreen.kt）実装
   - お気に入りフィルタ（FavoritesFilter.kt）実装
   - X Post埋め込み（XPostEmbed.kt）実装
   - フィルタ選択の永続化（PreferencesRepository.kt）実装

3. **UIテスト**
   - キャンペーン一覧表示のUIテスト
   - お気に入りフィルタのUIテスト
   - フィルタ選択永続化のテスト

4. **パフォーマンス最適化**
   - Firestore読み取り最適化（インデックス最適化）
   - 初期表示の高速化（遅延読み込み最適化）
   - モバイル4G環境で3秒以内の表示確認

5. **デプロイ**
   - Google Play Console設定
   - 内部テスト版リリース
   - 動作確認

## Dependencies

### 前提条件
- **001-admin-panel**: チェーン店とキャンペーンが管理画面で登録済み（前提条件、読み取り専用）
- **003-favorites**: お気に入りデータが必要（前提条件、お気に入りフィルタ機能）

### 提供するデータ
- **003-favorites**: お気に入り機能がキャンペーン一覧画面（002）を表示

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
  - WebViewのエラーハンドリング

### Risk 2: Firestore読み取り回数が想定超過
- **Impact**: コスト増加
- **Probability**: 低（Phase2では144回程度、150回以内）
- **Mitigation**:
  - Firestore読み取り最適化（インデックス最適化）
  - Firestoreキャッシュ活用
  - Firebase Consoleで監視

### Risk 3: 初期表示が遅い
- **Impact**: ユーザー体験悪化
- **Probability**: 中（X Post埋め込みの影響）
- **Mitigation**:
  - X Post埋め込みは非同期ロード
  - キャンペーン一覧は先に表示、X Postは後から表示
  - Skeleton UIで読み込み中を表示

### Risk 4: お気に入りチェーンが10件超過
- **Impact**: Firestoreの`whereIn`制約（最大10件）
- **Probability**: 低（Phase2では一般ユーザーがそこまで登録しない）
- **Mitigation**:
  - Phase2ではお気に入りチェーン10件までに制限
  - Phase3で複数回クエリに分割する実装に変更

## Success Metrics

### Phase2完了時の基準
- ✅ キャンペーン一覧の初期表示が3秒以内（モバイル4G環境、X Post埋め込み部分を除く）
- ✅ キャンペーンがスムーズにスクロールできる（10件以上ある場合）
- ✅ お気に入りフィルタの切り替えが1秒以内に反映される
- ✅ キャンペーンが0件の場合は正しい表示がされる（空リストではなく「データなし」と表示）
- ✅ フィルタ選択が次回訪問時も保持される確率が100%である（DataStoreで永続化）
- ✅ 単体テストカバレッジ70%以上
- ✅ UIテストが全て合格

### Phase3以降の目標
- 同時アクセス500人が可能な負荷テスト合格
- 初期表示2秒以内（フィルタ選択あり）
- リアルタイム更新機能
- パフォーマンス最適化（遅延読み込み）

## Notes

- **読み取り専用**: 002-campaign-listはキャンペーンの読み取り専用機能（書き込みは001-admin-panelと003-favoritesが担当）
- **001との整合性**: /campaigns、/chainsのスキーマは001-admin-panelと完全一致
- **003との依存関係**: お気に入りフィルタは003-favorites機能に依存（003が未完了の場合はフィルタ選択不可）
- **X Post埋め込み**: Phase2で商品画像の代わりに実装（代替画像なし）
- **1年経過フィルタ**: 販売開始日時から1年経過したキャンペーンは自動的に非表示（1年経過したキャンペーンは削除しない、非表示のみ）
- **Android優先**: Phase2ではAndroidアプリを優先（Webアプリ対応はPhase3以降）
