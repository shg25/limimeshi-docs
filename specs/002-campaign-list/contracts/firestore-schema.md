# Firestore Schema: キャンペーン一覧（Campaign List）

**Feature**: 002-campaign-list
**Date**: 2025-11-28
**Status**: Phase 1 - Contract
**Related**: [research.md](../research.md), [001-admin-panel Firestore Schema](../../001-admin-panel/contracts/firestore-schema.md)

## Overview

キャンペーン一覧機能（Androidアプリ）で使用するFirestoreコレクション構造（読み取り専用）

## Collections（読み取り専用）

### /campaigns/{campaignId}

期間限定キャンペーン（001-admin-panelで登録されたデータを読み取り）

```kotlin
data class Campaign(
    val id: String = "",               // Firestore自動生成ID（ドキュメントID）
    val chainId: String = "",          // 所属チェーンID（/chains/{chainId}への参照）
    val name: String = "",             // キャンペーン名
    val description: String? = null,   // 説明
    val xPostUrl: String? = null,      // X Post URL
    val saleStartTime: Timestamp = Timestamp.now(),  // 販売開始日時
    val saleEndTime: Timestamp? = null,              // 販売終了日時
    val createdAt: Timestamp = Timestamp.now(),      // 作成日時
    val updatedAt: Timestamp = Timestamp.now()       // 更新日時
)
```

**Field Details**:

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `chainId` | string | Yes | - | /chains/{chainId}への参照 |
| `name` | string | Yes | - | 最大200文字 |
| `description` | string | No | null | 最大1000文字 |
| `xPostUrl` | string | No | null | X Post URL形式 |
| `saleStartTime` | Timestamp | Yes | - | 販売開始日時 |
| `saleEndTime` | Timestamp | No | null | 販売終了日時（未定の場合はnull） |

**Queries Used**:

1. **全キャンペーン取得（1年以内、販売開始日時降順）**:
   ```kotlin
   val oneYearAgo = Timestamp(Date(System.currentTimeMillis() - 365L * 24 * 60 * 60 * 1000))

   db.collection("campaigns")
       .whereGreaterThanOrEqualTo("saleStartTime", oneYearAgo)
       .orderBy("saleStartTime", Query.Direction.DESCENDING)
       .get()
       .await()
   ```

2. **お気に入りフィルタ（chainIdの配列でフィルタ）**:
   ```kotlin
   db.collection("campaigns")
       .whereIn("chainId", favoriteChainIds)  // 最大10個
       .whereGreaterThanOrEqualTo("saleStartTime", oneYearAgo)
       .orderBy("saleStartTime", Query.Direction.DESCENDING)
       .get()
       .await()
   ```

**Indexes Required**:
- Single field index: `saleStartTime` (descending)
- Composite index: `chainId` (ascending) + `saleStartTime` (descending)

**Example Document**:
```json
{
  "chainId": "abc123",
  "name": "月見バーガー",
  "description": "ふんわりたまごとジューシーなビーフパティ",
  "xPostUrl": "https://twitter.com/McDonaldsJapan/status/1234567890",
  "saleStartTime": "2025-09-01T00:00:00.000Z",
  "saleEndTime": "2025-09-30T23:59:59.999Z",
  "createdAt": "2025-08-15T10:00:00.000Z",
  "updatedAt": "2025-08-15T10:00:00.000Z"
}
```

---

### /chains/{chainId}

チェーン店マスタ（001-admin-panelで登録されたデータを読み取り）

```kotlin
data class Chain(
    val id: String = "",               // Firestore自動生成ID（ドキュメントID）
    val name: String = "",             // チェーン名
    val furigana: String = "",         // ふりがな（50音順ソート用）
    val officialUrl: String? = null,   // 公式サイトURL
    val logoUrl: String? = null,       // ロゴ画像URL
    val favoriteCount: Int = 0,        // お気に入り登録数
    val createdAt: Timestamp = Timestamp.now(),  // 作成日時
    val updatedAt: Timestamp = Timestamp.now()   // 更新日時
)
```

**Queries Used**:

1. **チェーン名取得（キャンペーンのchainIdから）**:
   ```kotlin
   val chainDoc = db.collection("chains").document(campaign.chainId).get().await()
   val chainName = chainDoc.getString("name")
   ```

**Example Document**:
```json
{
  "name": "マクドナルド",
  "furigana": "まくどなるど",
  "officialUrl": "https://www.mcdonalds.co.jp/",
  "logoUrl": "https://example.com/logos/mcdonalds.png",
  "favoriteCount": 0,
  "createdAt": "2025-11-16T10:00:00.000Z",
  "updatedAt": "2025-11-16T10:00:00.000Z"
}
```

---

### /users/{userId}/favorites/{chainId}

お気に入りチェーン（003-favoritesで登録されたデータを読み取り）

```kotlin
data class Favorite(
    val chainId: String = "",          // お気に入り登録したチェーンID
    val createdAt: Timestamp = Timestamp.now()  // 登録日時
)
```

**Queries Used**:

1. **ログインユーザーのお気に入りチェーンID一覧**:
   ```kotlin
   val favoritesSnapshot = db.collection("users")
       .document(userId)
       .collection("favorites")
       .get()
       .await()

   val favoriteChainIds = favoritesSnapshot.documents.map { it.id }
   ```

**Example Document**:
```json
{
  "chainId": "abc123",
  "createdAt": "2025-11-18T10:00:00.000Z"
}
```

---

## Security Rules（読み取り専用）

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // キャンペーンコレクション（全ユーザー読み取り可能）
    match /campaigns/{campaignId} {
      allow read: if true;  // 全ユーザーが読み取り可能
      allow write: if false; // 002では書き込み不可（001で管理）
    }

    // チェーンコレクション（全ユーザー読み取り可能）
    match /chains/{chainId} {
      allow read: if true;  // 全ユーザーが読み取り可能
      allow write: if false; // 002では書き込み不可（001で管理）
    }

    // お気に入りコレクション（ログインユーザーのみ読み取り可能）
    match /users/{userId}/favorites/{chainId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if false; // 002では書き込み不可（003で管理）
    }
  }
}
```

---

## Performance Considerations

### Firestore読み取り回数（Phase2想定）

#### シナリオ1: 全キャンペーン表示（お気に入りフィルタOFF）
- **キャンペーン一覧取得**: 1回のクエリ（160件程度）= 160 reads
- **チェーン名取得**: メモリキャッシュ活用（16チェーン）= 0〜16 reads（初回のみ）
- **合計**: 160〜176 reads / 画面表示

#### シナリオ2: お気に入りフィルタON（お気に入りチェーン3件の場合）
- **お気に入りチェーンID一覧**: 1回のクエリ（3件）= 3 reads
- **キャンペーン一覧取得**: 1回のクエリ（約30件）= 30 reads
- **チェーン名取得**: メモリキャッシュ活用（3チェーン）= 0〜3 reads（初回のみ）
- **合計**: 33〜36 reads / 画面表示

### キャッシュ戦略
- **Firestoreキャッシュ**: デフォルトで有効、オフライン時は0 reads
- **チェーン名キャッシュ**: メモリ内でキャッシュ（Repository層）、2回目以降はFirestore読み取り不要

### コスト試算（Phase2想定）
- **DAU**: 100人
- **1日あたり画面表示**: 100人 × 3回 = 300回
- **1日あたり読み取り**: 300回 × 160 reads = 48,000 reads
- **月間読み取り**: 48,000 reads × 30日 = 1,440,000 reads
- **Firestore無料枠**: 50,000 reads / 日 = 1,500,000 reads / 月
- **Phase2コスト**: 無料枠内

---

## Data Model（Kotlin型定義）

```kotlin
import com.google.firebase.Timestamp

// キャンペーン
data class Campaign(
    val id: String = "",
    val chainId: String = "",
    val name: String = "",
    val description: String? = null,
    val xPostUrl: String? = null,
    val saleStartTime: Timestamp = Timestamp.now(),
    val saleEndTime: Timestamp? = null,
    val createdAt: Timestamp = Timestamp.now(),
    val updatedAt: Timestamp = Timestamp.now()
)

// チェーン店
data class Chain(
    val id: String = "",
    val name: String = "",
    val furigana: String = "",
    val officialUrl: String? = null,
    val logoUrl: String? = null,
    val favoriteCount: Int = 0,
    val createdAt: Timestamp = Timestamp.now(),
    val updatedAt: Timestamp = Timestamp.now()
)

// お気に入り
data class Favorite(
    val chainId: String = "",
    val createdAt: Timestamp = Timestamp.now()
)

// キャンペーンステータス（クライアント側で計算）
sealed class CampaignStatus {
    object Scheduled : CampaignStatus()  // 予定
    data class DaysElapsed(val days: Int) : CampaignStatus()  // 開始から◯日経過
    data class MonthsElapsed(val months: Int) : CampaignStatus()  // 開始から◯ヶ月以上経過
    object Ended : CampaignStatus()  // 終了

    fun toDisplayString(): String = when (this) {
        is Scheduled -> "予定"
        is DaysElapsed -> "開始から${days}日経過"
        is MonthsElapsed -> "開始から${months}ヶ月以上経過"
        is Ended -> "終了"
    }
}

// キャンペーン表示用（UIで使用）
data class CampaignWithChain(
    val campaign: Campaign,
    val chainName: String,
    val status: CampaignStatus
)
```

---

## Notes

- **読み取り専用**: 002-campaign-listはデータの読み取りのみ、書き込みは001-admin-panelと003-favoritesが担当
- **001との整合性**: /campaigns、/chainsのスキーマは001-admin-panelと完全一致
- **003との依存**: お気に入りフィルタは003-favorites機能に依存
- **キャッシュ活用**: Firestoreキャッシュとメモリキャッシュを併用してコスト削減
- **Android優先**: Phase2ではAndroidアプリを優先（Webアプリ対応はPhase3以降）
