# Data Model: 管理画面（Admin Panel）

**Feature**: 001-admin-panel
**Date**: 2025-11-16
**Status**: Phase 1 - Design
**Related**: [spec.md](./spec.md), [plan.md](./plan.md), [research.md](./research.md)

## Overview

管理画面で扱う3つの主要エンティティ（Chain、Menu、Admin）のデータモデル詳細設計

## Entities

### Chain（チェーン店）

```typescript
interface Chain {
  // 識別子
  id: string;                    // Firestore自動生成ID

  // 基本情報
  name: string;                  // チェーン名（必須、最大100文字）
  furigana: string;              // ふりがな（必須、最大100文字、ひらがなのみ）

  // 追加情報
  officialUrl?: string;          // 公式サイトURL（任意、URL形式）
  logoUrl?: string;              // ロゴ画像URL（任意、URL形式）

  // 集約データ
  favoriteCount: number;         // お気に入り登録数（デフォルト: 0）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（自動）
  updatedAt: Timestamp;          // 更新日時（自動）
}
```

**Validation Rules**:
- `name`: 必須、1〜100文字
- `furigana`: 必須、1〜100文字、ひらがなのみ（正規表現: `^[ぁ-ん]+$`）
- `officialUrl`: 任意、URL形式（https://で始まる）
- `logoUrl`: 任意、URL形式（https://で始まる）
- `favoriteCount`: 0以上の整数

**Indexes**:
- `furigana` (ascending): チェーン一覧を50音順でソート
- `favoriteCount` (descending): お気に入り登録数順でソート（Phase3以降）

**Business Rules**:
- ふりがなは50音順ソート用（チェーン一覧で使用）
- お気に入り登録数は集約データ（ユーザーがお気に入り登録・解除時に増減）
- Phase2では削除機能なし（将来的にカスケード削除を検討）

---

### Menu（メニュー）

```typescript
interface Menu {
  // 識別子
  id: string;                    // Firestore自動生成ID

  // 関連
  chainId: string;               // 所属チェーンID（外部キー、必須）

  // 基本情報
  name: string;                  // メニュー名（必須、最大100文字）
  description?: string;          // 説明（任意、最大500文字）
  price?: number;                // 価格（任意、0以上の整数）

  // 外部リンク
  xPostUrl?: string;             // X Post URL（任意、URL形式）

  // 販売期間
  saleStartTime: Timestamp;      // 販売開始日時（必須）
  saleEndTime?: Timestamp;       // 販売終了日時（任意）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（自動）
  updatedAt: Timestamp;          // 更新日時（自動）
}
```

**Computed Fields** (保存しない):
```typescript
interface MenuWithStatus extends Menu {
  status: 'scheduled' | 'days_elapsed' | 'months_elapsed' | 'ended';
  statusDisplay: string;         // 表示用文字列
  elapsedDays?: number;          // 発売からの経過日数
  elapsedMonths?: number;        // 発売からの経過月数
}
```

**Validation Rules**:
- `chainId`: 必須、`/chains/{chainId}` に存在すること
- `name`: 必須、1〜100文字
- `description`: 任意、最大500文字
- `price`: 任意、0以上の整数
- `xPostUrl`: 任意、URL形式（https://x.com/ または https://twitter.com/ で始まる）
- `saleStartTime`: 必須、ISO 8601形式
- `saleEndTime`: 任意、`saleStartTime` より後であること

**Indexes**:
- `chainId` (ascending): チェーン別メニュー一覧
- `saleStartTime` (descending): 販売開始日時の降順（メニュー一覧のデフォルトソート）
- Composite index: `chainId` (ascending) + `saleStartTime` (descending)

**Business Rules**:

1. **ステータス自動判定**（現在時刻基準）:
   - 予定: `current_time < saleStartTime`
   - 発売から◯日経過: `saleEndTime == null && (current_time - saleStartTime) < 1 month`
   - 発売から◯ヶ月以上経過: `saleEndTime == null && (current_time - saleStartTime) >= 1 month`
   - 販売終了: `current_time > saleEndTime`

2. **1年経過メニューの自動非表示**:
   - `saleStartTime` から1年以上経過したメニューは一覧に表示しない
   - データは削除せず、クエリで除外

3. **削除ルール**:
   - Phase2では削除機能あり（管理者が誤登録を修正可能）
   - チェーン店削除時のカスケード削除はPhase2では手動（管理者が先にメニューを削除）

---

### Admin（管理者）

```typescript
interface Admin {
  // 識別子（Firebase Authentication UID）
  id: string;                    // Firebase Auth UID

  // 基本情報
  displayName?: string;          // 表示名（任意、最大50文字）
  email: string;                 // メールアドレス（必須、Firebase Authから取得）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（自動）
}
```

**Custom Claims**:
```typescript
interface AdminCustomClaims {
  admin: true;                   // 管理者フラグ（必須）
}
```

**Validation Rules**:
- `email`: 必須、メールアドレス形式（Firebase Authで検証）
- `displayName`: 任意、最大50文字

**Business Rules**:
- Firebase Authentication の Custom Claims で管理者を識別
- Custom Claims: `{ admin: true }` を持つユーザーのみログイン可能
- Phase2では1名運用（複数管理者はPhase3以降）
- 管理者の追加・削除は Cloud Functions または Firebase CLI で実施

---

## Relationships

```
Chain (1) ────< (N) Menu
  ↓
  └─ favoriteCount: 集約データ（Favoriteエンティティから計算）

Admin ───> System (認証)
  ↓
  └─ Custom Claims で権限管理
```

**Entity Relationship Details**:

1. **Chain → Menu**:
   - 1対多の関係
   - `Menu.chainId` は `Chain.id` への参照
   - チェーン店削除時のカスケード削除はPhase2では手動

2. **Admin → System**:
   - Firebase Authentication で管理
   - Custom Claims で権限管理
   - Phase2では1名のみ

---

## State Transitions

### Menu Status Lifecycle

```
[管理者が登録]
      ↓
   予定 (current_time < saleStartTime)
      ↓
   [販売開始日時到達]
      ↓
   発売から◯日経過 (elapsed < 1 month)
      ↓
   [1ヶ月経過]
      ↓
   発売から◯ヶ月以上経過 (elapsed >= 1 month)
      ↓
   [販売終了日時到達 または 1年経過]
      ↓
   販売終了 / 自動非表示
```

**Transition Rules**:
- ステータスはリアルタイム計算（Firestoreには保存しない）
- 1年経過したメニューは自動的に一覧から除外
- 管理者が `saleEndTime` を設定すると「販売終了」に移行

---

## Data Integrity Rules

### Referential Integrity

- `Menu.chainId` は `/chains/{chainId}` に存在すること
- チェーン店削除時は関連メニューを先に削除（Phase2では手動、Phase3以降で自動化検討）

### Consistency Rules

1. **お気に入り登録数の一貫性**:
   - `Chain.favoriteCount` は Favorite エンティティの登録・解除時に増減
   - 集約データとして管理（リアルタイムカウントではない）

2. **タイムスタンプの一貫性**:
   - `createdAt` は作成時に自動設定（Firebase Server Timestamp）
   - `updatedAt` は更新時に自動設定（Firebase Server Timestamp）

### Validation at Write Time

- 必須フィールドのチェック
- 文字列長のチェック
- 日付の前後関係チェック（`saleEndTime > saleStartTime`）
- URL形式のチェック
- ふりがなの文字種チェック（ひらがなのみ）

---

## Performance Considerations

### Query Optimization

1. **メニュー一覧**:
   - `saleStartTime` (descending) でソート
   - 1年以上前のメニューを除外するクエリ
   - チェーン別フィルタは `chainId` インデックスを使用

2. **チェーン一覧**:
   - `furigana` (ascending) でソート（50音順）
   - お気に入り登録数順ソートは `favoriteCount` インデックスを使用（Phase3以降）

### Data Size

- Phase2想定: 16チェーン、平均160メニュー
- 1年で約2,000メニュー（16チェーン × 月10件 × 12ヶ月）
- 自動非表示により表示対象は常に2,000件以下

---

## Migration Strategy

Phase2では新規構築のため、マイグレーションは不要

Phase3以降で以下の変更を検討：
- Favorite エンティティの追加（お気に入り登録機能）
- Tag エンティティの追加（タグ管理機能）
- カスケード削除の自動化（Cloud Functions）
- リアルタイム更新対応（Firestore realtime listeners）

---

## Notes

- **Phase2 MVP範囲**: Chain、Menu、Admin のみ
- **Out of Scope**: Tag、Favorite（Phase3以降）
- **削除機能**: メニューのみ削除可能、チェーン店は削除不可（Phase2）
- **集約データ**: `favoriteCount` はPhase3で Favorite エンティティから計算
- **1年経過ルール**: データは削除せず、クエリで除外（監査ログとして保持）
