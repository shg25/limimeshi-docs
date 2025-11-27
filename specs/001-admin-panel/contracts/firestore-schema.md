# Firestore Schema: 管理画面（Admin Panel）

**Feature**: 001-admin-panel  
**Date**: 2025-11-16  
**Status**: Phase 1 - Contract  
**Related**: [data-model.md](../data-model.md), [react-admin-data-provider.md](./react-admin-data-provider.md)  

## Overview

管理画面で使用するFirestoreコレクション構造とセキュリティルール

## Collections

### /chains/{chainId}

チェーン店マスタ

```typescript
{
  // Firestore自動生成ID（ドキュメントID）
  id: string;

  // 基本情報
  name: string;                  // チェーン名
  furigana: string;              // ふりがな（50音順ソート用）

  // 追加情報
  officialUrl?: string;          // 公式サイトURL
  logoUrl?: string;              // ロゴ画像URL

  // 集約データ
  favoriteCount: number;         // お気に入り登録数（デフォルト: 0）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（Firebase Server Timestamp）
  updatedAt: Timestamp;          // 更新日時（Firebase Server Timestamp）
}
```

**Field Details**:

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `name` | string | Yes | - | 最大100文字 |
| `furigana` | string | Yes | - | 最大100文字、ひらがなのみ |
| `officialUrl` | string | No | null | URL形式（https://） |
| `logoUrl` | string | No | null | URL形式（https://） |
| `favoriteCount` | number | Yes | 0 | 0以上の整数 |
| `createdAt` | Timestamp | Yes | serverTimestamp() | 自動設定 |
| `updatedAt` | Timestamp | Yes | serverTimestamp() | 自動設定 |

**Indexes**:
- Single field index: `furigana` (ascending)
- Single field index: `favoriteCount` (descending) ※Phase3以降で使用

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

### /campaigns/{campaignId}

キャンペーン

```typescript
{
  // Firestore自動生成ID（ドキュメントID）
  id: string;

  // 関連
  chainId: string;               // 所属チェーンID（/chains/{chainId}への参照）

  // 基本情報
  name: string;                  // キャンペーン名
  description?: string;          // 説明

  // 外部リンク
  xPostUrl?: string;             // X Post URL

  // 販売期間
  saleStartTime: Timestamp;      // 販売開始日時
  saleEndTime?: Timestamp;       // 販売終了日時

  // メタデータ
  createdAt: Timestamp;          // 作成日時（Firebase Server Timestamp）
  updatedAt: Timestamp;          // 更新日時（Firebase Server Timestamp）
}
```

**Field Details**:

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `chainId` | string | Yes | - | `/chains/{chainId}` に存在すること |
| `name` | string | Yes | - | 最大100文字 |
| `description` | string | No | null | 最大500文字 |
| `xPostUrl` | string | No | null | URL形式（https://x.com/ または https://twitter.com/） |
| `saleStartTime` | Timestamp | Yes | - | ISO 8601形式 |
| `saleEndTime` | Timestamp | No | null | `saleStartTime` より後 |
| `createdAt` | Timestamp | Yes | serverTimestamp() | 自動設定 |
| `updatedAt` | Timestamp | Yes | serverTimestamp() | 自動設定 |

**Indexes**:
- Single field index: `chainId` (ascending)
- Single field index: `saleStartTime` (descending)
- Composite index: `chainId` (ascending) + `saleStartTime` (descending)

**Example Document**:
```json
{
  "chainId": "abc123",
  "name": "てりたま",
  "description": "たまごとテリヤキソースの期間限定キャンペーン",
  "xPostUrl": "https://x.com/McDonaldsJapan/status/1234567890",
  "saleStartTime": "2025-11-01T00:00:00.000Z",
  "saleEndTime": "2025-11-30T23:59:59.000Z",
  "createdAt": "2025-11-16T10:00:00.000Z",
  "updatedAt": "2025-11-16T10:00:00.000Z"
}
```

---

### /admins/{adminId}

管理者マスタ（Firebase Authentication UIDをドキュメントIDとして使用）

```typescript
{
  // Firebase Authentication UID（ドキュメントID）
  id: string;

  // 基本情報
  displayName?: string;          // 表示名
  email: string;                 // メールアドレス（Firebase Authから取得）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（Firebase Server Timestamp）
}
```

**Field Details**:

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `displayName` | string | No | null | 最大50文字 |
| `email` | string | Yes | - | メールアドレス形式（Firebase Authから取得） |
| `createdAt` | Timestamp | Yes | serverTimestamp() | 自動設定 |

**Custom Claims** (Firebase Authentication):
```json
{
  "admin": true
}
```

**Example Document**:
```json
{
  "displayName": "重次弘規",
  "email": "admin@limimeshi.com",
  "createdAt": "2025-11-16T10:00:00.000Z"
}
```

---

## Security Rules

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 管理者判定関数
    function isAdmin() {
      return request.auth != null && request.auth.token.admin == true;
    }

    // タイムスタンプ自動設定関数
    function hasValidTimestamps() {
      return request.resource.data.createdAt == request.time &&
             request.resource.data.updatedAt == request.time;
    }

    function hasValidUpdateTimestamp() {
      return request.resource.data.updatedAt == request.time;
    }

    // チェーン店
    match /chains/{chainId} {
      // 管理者のみ読み書き可能
      allow read: if isAdmin();
      allow create: if isAdmin() && hasValidTimestamps();
      allow update: if isAdmin() && hasValidUpdateTimestamp();
      // Phase2では削除機能なし
      allow delete: if false;
    }

    // キャンペーン
    match /campaigns/{campaignId} {
      // 管理者のみ読み書き可能
      allow read: if isAdmin();
      allow create: if isAdmin() &&
                       hasValidTimestamps() &&
                       exists(/databases/$(database)/documents/chains/$(request.resource.data.chainId));
      allow update: if isAdmin() &&
                       hasValidUpdateTimestamp() &&
                       exists(/databases/$(database)/documents/chains/$(request.resource.data.chainId));
      allow delete: if isAdmin();
    }

    // 管理者
    match /admins/{adminId} {
      // 管理者のみ読み取り可能
      allow read: if isAdmin();
      // 作成・更新・削除は Cloud Functions または Firebase CLI のみ
      allow write: if false;
    }
  }
}
```

**Security Rules Explanation**:

1. **管理者のみアクセス可能**:
   - `isAdmin()` 関数で Custom Claims `{ admin: true }` をチェック
   - 未ログインユーザー、一般ユーザーはアクセス不可

2. **タイムスタンプ自動設定の強制**:
   - `createdAt` は作成時に `request.time`（サーバー時刻）を設定
   - `updatedAt` は作成・更新時に `request.time` を設定
   - クライアント側で任意のタイムスタンプを設定不可

3. **外部キー整合性チェック**:
   - `Campaign.chainId` は `/chains/{chainId}` に存在することを確認
   - `exists()` 関数で参照整合性を保証

4. **Phase2の制約**:
   - チェーン店削除は禁止（`allow delete: if false`）
   - 管理者の作成・更新・削除は Cloud Functions または Firebase CLI のみ

---

## Firestore Indexes

### firestore.indexes.json

```json
{
  "indexes": [
    {
      "collectionGroup": "campaigns",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "chainId",
          "order": "ASCENDING"
        },
        {
          "fieldPath": "saleStartTime",
          "order": "DESCENDING"
        }
      ]
    }
  ],
  "fieldOverrides": []
}
```

**Index Explanation**:
- **Composite index**: `chainId` (ascending) + `saleStartTime` (descending)
- **Use case**: チェーン別キャンペーン一覧を販売開始日時の降順でソート
- **Example query**:
  ```typescript
  db.collection('campaigns')
    .where('chainId', '==', 'abc123')
    .orderBy('saleStartTime', 'desc')
  ```

---

## Query Examples

### チェーン一覧を50音順で取得

```typescript
const chains = await db.collection('chains')
  .orderBy('furigana', 'asc')
  .get();
```

### キャンペーン一覧を販売開始日時の降順で取得（1年以内のみ）

```typescript
const oneYearAgo = new Date();
oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

const campaigns = await db.collection('campaigns')
  .where('saleStartTime', '>=', oneYearAgo)
  .orderBy('saleStartTime', 'desc')
  .get();
```

### 特定チェーンのキャンペーン一覧を取得

```typescript
const campaigns = await db.collection('campaigns')
  .where('chainId', '==', 'abc123')
  .orderBy('saleStartTime', 'desc')
  .get();
```

### 管理者情報を取得

```typescript
const admin = await db.collection('admins')
  .doc(firebaseAuth.currentUser.uid)
  .get();
```

---

## Data Migration

Phase2では新規構築のため、マイグレーションは不要

Phase3以降で以下のコレクションを追加予定：
- `/users/{userId}/favorites`: お気に入り登録
- `/tags/{tagId}`: タグマスタ
- `/campaign_tags/{campaignTagId}`: キャンペーンタグ中間テーブル

---

## Notes

- **Phase2 MVP範囲**: `/chains`, `/campaigns`, `/admins` のみ
- **Out of Scope**: `/users/{userId}/favorites`, `/tags`, `/campaign_tags`（Phase3以降）
- **削除機能**: キャンペーンのみ削除可能、チェーン店は削除不可（Phase2）
- **Custom Claims**: Firebase Authentication の Custom Claims で管理者を識別
- **セキュリティルール**: 管理者以外は一切アクセス不可
- **タイムスタンプ**: Firebase Server Timestamp を使用（クライアント時刻非依存）
