# Firestoreインデックス設計

**作成日**: 2025-11-19  
**ベストプラクティス**: [firestore-best-practices.md](./firestore-best-practices.md)  
**関連仕様**: [002-chain-list](../specs/002-chain-list/), [003-favorites](../specs/003-favorites/)  
  
---

## 概要

本プロジェクトで必要なFirestoreインデックス。クエリパターンに基づいて最適化されている。

### インデックスの種類

| 種類 | 説明 | 作成方法 |
|------|------|----------|
| **単一フィールドインデックス** | 1フィールドのソート・フィルタ | 自動作成 |
| **複合インデックス** | 複数フィールドの組み合わせ | 手動作成 |

---

## 必要なインデックス一覧

### 複合インデックス（手動作成）

| コレクション | フィールド | 用途 |
|-------------|-----------|------|
| /campaigns | `chainId` (asc) + `saleStartTime` (desc) | お気に入りフィルタ+新しい順ソート |

### 単一フィールドインデックス（自動作成）

| コレクション | フィールド | 用途 |
|-------------|-----------|------|
| /campaigns | `saleStartTime` (desc) | 新しい順ソート |
| /chains | `furigana` (asc) | 50音順ソート |

---

## 詳細設計

### /campaigns コレクション

#### 複合インデックス: chainId + saleStartTime

**用途**: お気に入りチェーンフィルタ + 販売開始日時降順ソート

**クエリ例**:
```typescript
// お気に入りフィルタON（003-favoritesのchainIdリストを使用）
const oneYearAgo = Timestamp.fromDate(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000));

const q = query(
  collection(db, 'campaigns'),
  where('chainId', 'in', favoriteChainIds),  // 最大10件
  where('saleStartTime', '>=', oneYearAgo),
  orderBy('saleStartTime', 'desc')
);
```

**インデックス定義**:
```json
{
  "collectionGroup": "campaigns",
  "queryScope": "COLLECTION",
  "fields": [
    { "fieldPath": "chainId", "order": "ASCENDING" },
    { "fieldPath": "saleStartTime", "order": "DESCENDING" }
  ]
}
```

**必要な理由**:
- `where('chainId', 'in', ...)` と `orderBy('saleStartTime', 'desc')` を同時に使用
- 単一フィールドインデックスでは対応できない

#### 単一フィールドインデックス: saleStartTime (desc)

**用途**: 全キャンペーン取得（新しい順）

**クエリ例**:
```typescript
// お気に入りフィルタOFF（全キャンペーン）
const q = query(
  collection(db, 'campaigns'),
  where('saleStartTime', '>=', oneYearAgo),
  orderBy('saleStartTime', 'desc')
);
```

**必要な理由**:
- 自動作成されるが、降順インデックスが必要
- `where` + `orderBy` の組み合わせで使用

---

### /chains コレクション

#### 単一フィールドインデックス: furigana (asc)

**用途**: チェーン店一覧（50音順）

**クエリ例**:
```typescript
// 管理画面でのチェーン店一覧
const q = query(
  collection(db, 'chains'),
  orderBy('furigana', 'asc')
);
```

**必要な理由**:
- 自動作成される
- 管理画面での一覧表示に使用

---

## インデックス除外設定

以下のフィールドはクエリで使用しないため、インデックスを除外する。

| コレクション | フィールド | 除外理由 |
|-------------|-----------|---------|
| /campaigns | description | 検索に使用しない、長文字列 |
| /campaigns | xPostUrl | 検索に使用しない |
| /chains | officialUrl | 検索に使用しない |
| /chains | logoUrl | 検索に使用しない |

**除外設定方法**:
Firebase Consoleで「インデックス除外」を設定、またはfirestore.indexes.jsonで定義。

---

## firestore.indexes.json

```json
{
  "indexes": [
    {
      "collectionGroup": "campaigns",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "chainId", "order": "ASCENDING" },
        { "fieldPath": "saleStartTime", "order": "DESCENDING" }
      ]
    }
  ],
  "fieldOverrides": [
    {
      "collectionGroup": "campaigns",
      "fieldPath": "description",
      "indexes": []
    },
    {
      "collectionGroup": "campaigns",
      "fieldPath": "xPostUrl",
      "indexes": []
    },
    {
      "collectionGroup": "chains",
      "fieldPath": "officialUrl",
      "indexes": []
    },
    {
      "collectionGroup": "chains",
      "fieldPath": "logoUrl",
      "indexes": []
    }
  ]
}
```

---

## インデックス管理

### 作成方法

#### 1. Firebase Console

1. Firebase Console → Firestore Database → インデックス
2. 「複合インデックスを追加」をクリック
3. コレクション、フィールド、順序を設定
4. 「作成」をクリック

#### 2. Firebase CLI

```bash
# インデックスをデプロイ
firebase deploy --only firestore:indexes
```

#### 3. エラーメッセージから作成

クエリ実行時にインデックスが不足している場合、エラーメッセージにインデックス作成用のリンクが表示される。リンクをクリックするだけで作成可能。

### 構築時間

- **セットアップ時間**: 数分
- **バックフィル時間**: 既存データ量に依存
- Phase2の160件程度であれば数分で完了

### 監視

```bash
# インデックス構築状況を確認
gcloud firestore indexes list --project=limimeshi-dev
```

---

## Phase2でのインデックス要件

| インデックス | 必須/推奨 | 理由 |
|-------------|---------|------|
| campaigns: chainId + saleStartTime | **必須** | お気に入りフィルタ機能に必要 |
| campaigns: saleStartTime (desc) | 自動 | 全キャンペーン取得に使用 |
| chains: furigana (asc) | 自動 | 管理画面一覧に使用 |

---

## コスト考慮

### インデックスのストレージコスト

- 各インデックスはストレージを消費
- Phase2の規模（160キャンペーン、16チェーン）では無視できるレベル
- 不要なインデックスは除外設定で削減

### インデックス構築の注意

- 大量データがある状態でインデックスを追加すると、バックフィル処理に時間がかかる
- Phase2開始前にインデックスを作成しておくことを推奨

---

## 参考

- [Firestore設計ベストプラクティス](./firestore-best-practices.md)
- [インデックスの管理（公式）](https://firebase.google.com/docs/firestore/query-data/indexing)
- [インデックスの制限（公式）](https://firebase.google.com/docs/firestore/quotas#indexes)
