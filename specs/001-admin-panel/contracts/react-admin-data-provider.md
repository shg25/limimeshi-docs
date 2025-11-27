# React Admin Data Provider Interface: 管理画面（Admin Panel）

**Feature**: 001-admin-panel  
**Date**: 2025-11-16  
**Status**: Phase 1 - Contract  
**Related**: [data-model.md](../data-model.md), [firestore-schema.md](./firestore-schema.md)  

## Overview

React Admin と Firestore の統合インターフェース仕様

## Data Provider Interface

### 概要

React Admin の Data Provider は以下のメソッドを実装する必要がある：

```typescript
interface DataProvider {
  getList:    (resource, params) => Promise<{ data: Record[], total: number }>
  getOne:     (resource, params) => Promise<{ data: Record }>
  getMany:    (resource, params) => Promise<{ data: Record[] }>
  getManyReference: (resource, params) => Promise<{ data: Record[], total: number }>
  create:     (resource, params) => Promise<{ data: Record }>
  update:     (resource, params) => Promise<{ data: Record }>
  updateMany: (resource, params) => Promise<{ data: any[] }>
  delete:     (resource, params) => Promise<{ data: Record }>
  deleteMany: (resource, params) => Promise<{ data: any[] }>
}
```

本プロジェクトでは `react-admin-firebase` パッケージを使用し、一部のメソッドをカスタマイズする

---

## Resource: chains

### getList

チェーン一覧を取得

**Request**:
```typescript
{
  pagination: { page: 1, perPage: 25 },
  sort: { field: 'furigana', order: 'ASC' },
  filter: {}
}
```

**Response**:
```typescript
{
  data: [
    {
      id: 'abc123',
      name: 'マクドナルド',
      furigana: 'まくどなるど',
      officialUrl: 'https://www.mcdonalds.co.jp/',
      logoUrl: 'https://example.com/logos/mcdonalds.png',
      favoriteCount: 0,
      createdAt: '2025-11-16T10:00:00.000Z',
      updatedAt: '2025-11-16T10:00:00.000Z'
    },
    // ...
  ],
  total: 16
}
```

**Firestore Query**:
```typescript
db.collection('chains')
  .orderBy('furigana', 'asc')
  .limit(25)
```

---

### getOne

特定のチェーン店を取得

**Request**:
```typescript
{
  id: 'abc123'
}
```

**Response**:
```typescript
{
  data: {
    id: 'abc123',
    name: 'マクドナルド',
    furigana: 'まくどなるど',
    officialUrl: 'https://www.mcdonalds.co.jp/',
    logoUrl: 'https://example.com/logos/mcdonalds.png',
    favoriteCount: 0,
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:00:00.000Z'
  }
}
```

**Firestore Query**:
```typescript
db.collection('chains').doc('abc123').get()
```

---

### create

新しいチェーン店を作成

**Request**:
```typescript
{
  data: {
    name: 'マクドナルド',
    furigana: 'まくどなるど',
    officialUrl: 'https://www.mcdonalds.co.jp/',
    logoUrl: 'https://example.com/logos/mcdonalds.png'
  }
}
```

**Response**:
```typescript
{
  data: {
    id: 'auto-generated-id',
    name: 'マクドナルド',
    furigana: 'まくどなるど',
    officialUrl: 'https://www.mcdonalds.co.jp/',
    logoUrl: 'https://example.com/logos/mcdonalds.png',
    favoriteCount: 0,  // デフォルト値
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:00:00.000Z'
  }
}
```

**Firestore Operation**:
```typescript
const docRef = db.collection('chains').doc();
await docRef.set({
  name: data.name,
  furigana: data.furigana,
  officialUrl: data.officialUrl || null,
  logoUrl: data.logoUrl || null,
  favoriteCount: 0,
  createdAt: FieldValue.serverTimestamp(),
  updatedAt: FieldValue.serverTimestamp()
});
```

**Validation**:
- `name`: 必須、1〜100文字
- `furigana`: 必須、1〜100文字、ひらがなのみ（`^[ぁ-ん]+$`）
- `officialUrl`: 任意、URL形式（https://）
- `logoUrl`: 任意、URL形式（https://）

---

### update

既存のチェーン店を更新

**Request**:
```typescript
{
  id: 'abc123',
  data: {
    name: 'マクドナルド（更新）',
    officialUrl: 'https://www.mcdonalds.co.jp/'
  },
  previousData: { ... }
}
```

**Response**:
```typescript
{
  data: {
    id: 'abc123',
    name: 'マクドナルド（更新）',
    furigana: 'まくどなるど',
    officialUrl: 'https://www.mcdonalds.co.jp/',
    logoUrl: 'https://example.com/logos/mcdonalds.png',
    favoriteCount: 0,
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:30:00.000Z'
  }
}
```

**Firestore Operation**:
```typescript
await db.collection('chains').doc('abc123').update({
  name: data.name,
  officialUrl: data.officialUrl,
  updatedAt: FieldValue.serverTimestamp()
});
```

**Notes**:
- `createdAt` は変更しない
- `favoriteCount` は自動計算フィールドのため変更不可（Phase3でFavoriteエンティティから計算）

---

### delete

**Phase2では実装しない**

チェーン店の削除機能はPhase2では提供しない

```typescript
delete: (resource, params) => {
  if (resource === 'chains') {
    throw new Error('チェーン店の削除はPhase2では未対応です');
  }
}
```

---

## Resource: campaigns

### getList

キャンペーン一覧を取得（1年以内のキャンペーンのみ）

**Request**:
```typescript
{
  pagination: { page: 1, perPage: 25 },
  sort: { field: 'saleStartTime', order: 'DESC' },
  filter: { chainId: 'abc123' }  // 任意
}
```

**Response**:
```typescript
{
  data: [
    {
      id: 'campaign123',
      chainId: 'abc123',
      name: 'てりたま',
      description: 'たまごとテリヤキソースの期間限定キャンペーン',
      xPostUrl: 'https://x.com/McDonaldsJapan/status/1234567890',
      saleStartTime: '2025-11-01T00:00:00.000Z',
      saleEndTime: '2025-11-30T23:59:59.000Z',
      createdAt: '2025-11-16T10:00:00.000Z',
      updatedAt: '2025-11-16T10:00:00.000Z',

      // 計算フィールド（Firestoreには保存しない）
      status: 'scheduled',
      statusDisplay: '予定',
      elapsedDays: null
    },
    // ...
  ],
  total: 160
}
```

**Firestore Query**:
```typescript
const oneYearAgo = new Date();
oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

let query = db.collection('campaigns')
  .where('saleStartTime', '>=', oneYearAgo)
  .orderBy('saleStartTime', 'desc');

// チェーン別フィルタ（任意）
if (filter.chainId) {
  query = query.where('chainId', '==', filter.chainId);
}

const snapshot = await query.limit(25).get();
```

**Status Calculation**:
```typescript
function calculateStatus(campaign: Campaign): CampaignWithStatus {
  const now = new Date();
  const saleStart = new Date(campaign.saleStartTime);
  const saleEnd = campaign.saleEndTime ? new Date(campaign.saleEndTime) : null;

  // 予定
  if (now < saleStart) {
    return {
      ...campaign,
      status: 'scheduled',
      statusDisplay: '予定',
      elapsedDays: null
    };
  }

  // 終了
  if (saleEnd && now > saleEnd) {
    return {
      ...campaign,
      status: 'ended',
      statusDisplay: '終了',
      elapsedDays: null
    };
  }

  // 開始からの経過日数
  const elapsedMs = now.getTime() - saleStart.getTime();
  const elapsedDays = Math.floor(elapsedMs / (1000 * 60 * 60 * 24));

  // 開始から◯日経過（1ヶ月未満）
  if (elapsedDays < 30) {
    return {
      ...campaign,
      status: 'days_elapsed',
      statusDisplay: `開始から${elapsedDays}日経過`,
      elapsedDays
    };
  }

  // 開始から◯ヶ月以上経過
  const elapsedMonths = Math.floor(elapsedDays / 30);
  return {
    ...campaign,
    status: 'months_elapsed',
    statusDisplay: `開始から${elapsedMonths}ヶ月以上経過`,
    elapsedDays,
    elapsedMonths
  };
}
```

---

### getOne

特定のキャンペーンを取得

**Request**:
```typescript
{
  id: 'campaign123'
}
```

**Response**:
```typescript
{
  data: {
    id: 'campaign123',
    chainId: 'abc123',
    name: 'てりたま',
    description: 'たまごとテリヤキソースの期間限定キャンペーン',
    xPostUrl: 'https://x.com/McDonaldsJapan/status/1234567890',
    saleStartTime: '2025-11-01T00:00:00.000Z',
    saleEndTime: '2025-11-30T23:59:59.000Z',
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:00:00.000Z',

    // 計算フィールド
    status: 'scheduled',
    statusDisplay: '予定',
    elapsedDays: null
  }
}
```

**Firestore Query**:
```typescript
const doc = await db.collection('campaigns').doc('campaign123').get();
const campaign = doc.data();
return calculateStatus(campaign);
```

---

### getManyReference

特定チェーンのキャンペーン一覧を取得

**Request**:
```typescript
{
  target: 'chainId',
  id: 'abc123',
  pagination: { page: 1, perPage: 25 },
  sort: { field: 'saleStartTime', order: 'DESC' },
  filter: {}
}
```

**Response**:
```typescript
{
  data: [ /* キャンペーン一覧 */ ],
  total: 10
}
```

**Firestore Query**:
```typescript
db.collection('campaigns')
  .where('chainId', '==', 'abc123')
  .orderBy('saleStartTime', 'desc')
  .limit(25)
```

**Use Case**:
- React Admin の `<ReferenceManyField>` で使用
- チェーン詳細ページでキャンペーン一覧を表示

---

### create

新しいキャンペーンを作成

**Request**:
```typescript
{
  data: {
    chainId: 'abc123',
    name: 'てりたま',
    description: 'たまごとテリヤキソースの期間限定キャンペーン',
    xPostUrl: 'https://x.com/McDonaldsJapan/status/1234567890',
    saleStartTime: '2025-11-01T00:00:00.000Z',
    saleEndTime: '2025-11-30T23:59:59.000Z'
  }
}
```

**Response**:
```typescript
{
  data: {
    id: 'auto-generated-id',
    chainId: 'abc123',
    name: 'てりたま',
    description: 'たまごとテリヤキソースの期間限定キャンペーン',
    xPostUrl: 'https://x.com/McDonaldsJapan/status/1234567890',
    saleStartTime: '2025-11-01T00:00:00.000Z',
    saleEndTime: '2025-11-30T23:59:59.000Z',
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:00:00.000Z',

    // 計算フィールド
    status: 'scheduled',
    statusDisplay: '予定',
    elapsedDays: null
  }
}
```

**Firestore Operation**:
```typescript
const docRef = db.collection('campaigns').doc();
await docRef.set({
  chainId: data.chainId,
  name: data.name,
  description: data.description || null,
  xPostUrl: data.xPostUrl || null,
  saleStartTime: data.saleStartTime,
  saleEndTime: data.saleEndTime || null,
  createdAt: FieldValue.serverTimestamp(),
  updatedAt: FieldValue.serverTimestamp()
});
```

**Validation**:
- `chainId`: 必須、`/chains/{chainId}` に存在すること
- `name`: 必須、1〜100文字
- `description`: 任意、最大500文字
- `xPostUrl`: 任意、URL形式（https://x.com/ または https://twitter.com/）
- `saleStartTime`: 必須、ISO 8601形式
- `saleEndTime`: 任意、`saleStartTime` より後であること

---

### update

既存のキャンペーンを更新

**Request**:
```typescript
{
  id: 'campaign123',
  data: {
    name: 'てりたま（更新）',
  },
  previousData: { ... }
}
```

**Response**:
```typescript
{
  data: {
    id: 'campaign123',
    chainId: 'abc123',
    name: 'てりたま（更新）',
    description: 'たまごとテリヤキソースの期間限定キャンペーン',
    xPostUrl: 'https://x.com/McDonaldsJapan/status/1234567890',
    saleStartTime: '2025-11-01T00:00:00.000Z',
    saleEndTime: '2025-11-30T23:59:59.000Z',
    createdAt: '2025-11-16T10:00:00.000Z',
    updatedAt: '2025-11-16T10:30:00.000Z',

    // 計算フィールド
    status: 'scheduled',
    statusDisplay: '予定',
    elapsedDays: null
  }
}
```

**Firestore Operation**:
```typescript
await db.collection('campaigns').doc('campaign123').update({
  name: data.name,
  updatedAt: FieldValue.serverTimestamp()
});
```

**Notes**:
- `createdAt` は変更しない
- `chainId` の変更は可能（別チェーンに移動）

---

### delete

キャンペーンを削除

**Request**:
```typescript
{
  id: 'campaign123',
  previousData: { ... }
}
```

**Response**:
```typescript
{
  data: {
    id: 'campaign123'
  }
}
```

**Firestore Operation**:
```typescript
await db.collection('campaigns').doc('campaign123').delete();
```

**Notes**:
- Phase2では削除機能あり（管理者が誤登録を修正可能）

---

## Resource: admins

### getList

管理者一覧を取得

**Request**:
```typescript
{
  pagination: { page: 1, perPage: 25 },
  sort: { field: 'createdAt', order: 'DESC' },
  filter: {}
}
```

**Response**:
```typescript
{
  data: [
    {
      id: 'firebase-auth-uid',
      displayName: '重次弘規',
      email: 'admin@limimeshi.com',
      createdAt: '2025-11-16T10:00:00.000Z'
    }
  ],
  total: 1
}
```

**Firestore Query**:
```typescript
db.collection('admins')
  .orderBy('createdAt', 'desc')
  .limit(25)
```

---

### getOne

特定の管理者を取得

**Request**:
```typescript
{
  id: 'firebase-auth-uid'
}
```

**Response**:
```typescript
{
  data: {
    id: 'firebase-auth-uid',
    displayName: '重次弘規',
    email: 'admin@limimeshi.com',
    createdAt: '2025-11-16T10:00:00.000Z'
  }
}
```

**Firestore Query**:
```typescript
db.collection('admins').doc('firebase-auth-uid').get()
```

---

### create, update, delete

**Phase2では実装しない**

管理者の作成・更新・削除は Cloud Functions または Firebase CLI で実施

```typescript
create: (resource, params) => {
  if (resource === 'admins') {
    throw new Error('管理者の作成はCloud FunctionsまたはFirebase CLIで実施してください');
  }
}
```

---

## Error Handling

### Validation Errors

```typescript
{
  status: 400,
  body: {
    message: 'Validation failed',
    errors: {
      furigana: 'ひらがなのみ入力してください',
      saleEndTime: '販売終了日時は販売開始日時より後に設定してください'
    }
  }
}
```

### Not Found Errors

```typescript
{
  status: 404,
  body: {
    message: 'チェーン店が見つかりません'
  }
}
```

### Permission Errors

```typescript
{
  status: 403,
  body: {
    message: '管理者権限が必要です'
  }
}
```

---

## Notes

- **Phase2 MVP範囲**: chains、campaigns、admins のみ
- **Out of Scope**: favorites、tags、campaign_tags（Phase3以降）
- **計算フィールド**: `status` と `statusDisplay` は getList/getOne で追加（Firestoreには保存しない）
- **削除機能**: キャンペーンのみ削除可能、チェーン店・管理者は削除不可（Phase2）
- **react-admin-firebase**: `react-admin-firebase` パッケージを使用し、一部カスタマイズ
- **タイムスタンプ**: Firebase Server Timestamp を使用（クライアント時刻非依存）
