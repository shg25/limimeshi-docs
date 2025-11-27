# Firestoreコレクション設計

**作成日**: 2025-11-19  
**ベストプラクティス**: [firestore-best-practices.md](./firestore-best-practices.md)  
**関連仕様**: [001-admin-panel](../specs/001-admin-panel/), [002-chain-list](../specs/002-chain-list/), [003-favorites](../specs/003-favorites/)  
  
---

## 概要

本プロジェクトで使用するFirestoreコレクションの設計。アクセスパターンに基づいて最適化されている。

### コレクション一覧

| コレクション | 用途 | 担当機能 | Phase |
|-------------|------|---------|-------|
| `/chains` | チェーン店マスタ | 001-admin-panel | Phase2 |
| `/campaigns` | キャンペーン | 001-admin-panel | Phase2 |
| `/admins` | 管理者情報 | 001-admin-panel | Phase2 |
| `/users/{userId}/favorites` | お気に入りチェーン | 003-favorites | Phase2 |
| `/reviews` | レビュー | - | Phase3以降 |

### データフロー

```
管理者 → 001-admin-panel → /chains, /campaigns (書き込み)
                              ↓
一般ユーザー → 002-chain-list → /chains, /campaigns (読み取り)
                              ↓
ログインユーザー → 003-favorites → /users/{userId}/favorites (読み書き)
                                   ↓
                              /chains.favoriteCount (集約更新)
```

---

## /chains（チェーン店マスタ）

### 概要

大手チェーン店の基本情報を管理。Phase2では16店舗を想定。

### スキーマ

```typescript
interface Chain {
  // ドキュメントID（Firestore自動生成）
  id: string;

  // 基本情報
  name: string;                  // チェーン名（必須、最大100文字）
  furigana: string;              // ふりがな（必須、最大100文字、50音順ソート用）

  // 追加情報
  officialUrl?: string;          // 公式サイトURL（任意）
  logoUrl?: string;              // ロゴ画像URL（任意）

  // 集約データ
  favoriteCount: number;         // お気に入り登録数（デフォルト: 0）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（serverTimestamp()）
  updatedAt: Timestamp;          // 更新日時（serverTimestamp()）
}
```

### フィールド詳細

| フィールド | 型 | 必須 | デフォルト | バリデーション |
|-----------|-----|------|-----------|---------------|
| name | string | Yes | - | 1〜100文字 |
| furigana | string | Yes | - | 1〜100文字、ひらがなのみ |
| officialUrl | string | No | null | URL形式 |
| logoUrl | string | No | null | URL形式 |
| favoriteCount | number | Yes | 0 | 0以上の整数 |
| createdAt | Timestamp | Yes | serverTimestamp() | - |
| updatedAt | Timestamp | Yes | serverTimestamp() | - |

### アクセスパターン

| パターン | クエリ | 用途 | 頻度 |
|---------|-------|------|------|
| 全件取得（50音順） | `orderBy('furigana', 'asc')` | 管理画面一覧 | 低 |
| ID指定取得 | `doc(db, 'chains', chainId)` | メニュー表示時のチェーン名取得 | 高 |
| 検索 | `where('name', '>=', keyword)` | 管理画面検索 | 低 |

### 設計判断

- **トップレベルコレクション**：メニューとの独立性、クロスコレクションクエリ不要
- **favoriteCount埋め込み**：003-favoritesでTransactionにより更新、読み取り最適化
- **削除機能なし**：Phase2では16店舗固定、誤削除リスク回避

---

## /campaigns（キャンペーン）

### 概要

キャンペーンの情報を管理。001-admin-panelで登録、002-chain-listで表示。

### スキーマ

```typescript
interface Campaign {
  // ドキュメントID（Firestore自動生成）
  id: string;

  // 関連
  chainId: string;               // 所属チェーンID（/chains/{chainId}への参照）

  // 基本情報
  name: string;                  // キャンペーン名（必須、最大100文字）
  description?: string;          // 説明（任意、最大500文字、キャンペーン内のメニュー詳細などを記載可能）

  // 外部リンク
  xPostUrl?: string;             // X Post URL（任意、代表的な1投稿）

  // 販売期間
  saleStartTime: Timestamp;      // 販売開始日時（必須）
  saleEndTime?: Timestamp;       // 販売終了日時（任意、未定の場合はnull）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（serverTimestamp()）
  updatedAt: Timestamp;          // 更新日時（serverTimestamp()）
}
```

### フィールド詳細

| フィールド | 型 | 必須 | デフォルト | バリデーション |
|-----------|-----|------|-----------|---------------|
| chainId | string | Yes | - | /chains/{chainId}に存在すること |
| name | string | Yes | - | 1〜100文字 |
| description | string | No | null | 最大500文字 |
| xPostUrl | string | No | null | X Post URL形式 |
| saleStartTime | Timestamp | Yes | - | ISO 8601形式 |
| saleEndTime | Timestamp | No | null | saleStartTimeより後 |
| createdAt | Timestamp | Yes | serverTimestamp() | - |
| updatedAt | Timestamp | Yes | serverTimestamp() | - |

### アクセスパターン

| パターン | クエリ | 用途 | 頻度 |
|---------|-------|------|------|
| 1年以内（新しい順） | `where('saleStartTime', '>=', oneYearAgo).orderBy('saleStartTime', 'desc')` | キャンペーン一覧 | 高 |
| お気に入りフィルタ | `where('chainId', 'in', [...]).where('saleStartTime', '>=', oneYearAgo).orderBy(...)` | お気に入りチェーンのキャンペーン | 中 |
| チェーン別一覧 | `where('chainId', '==', chainId).orderBy('saleStartTime', 'desc')` | 管理画面 | 低 |
| 検索 | `where('name', '>=', keyword)` | 管理画面検索 | 低 |

### 設計判断

- **トップレベルコレクション**：チェーンのサブコレクションにしない（クロスチェーンクエリが必要）
- **chainIdフィールド**：参照ではなく文字列（クエリ簡素化）
- **ステータスは保存しない**：saleStartTime/saleEndTimeから計算（クライアント側）

---

## /admins（管理者情報）

### 概要

管理者の追加情報を管理。認証はFirebase Authentication、権限はCustom Claims。

### スキーマ

```typescript
interface Admin {
  // ドキュメントID = Firebase Authentication UID
  id: string;

  // 基本情報
  displayName?: string;          // 表示名（任意）
  email: string;                 // メールアドレス（必須）

  // メタデータ
  createdAt: Timestamp;          // 作成日時（serverTimestamp()）
}
```

### フィールド詳細

| フィールド | 型 | 必須 | デフォルト | バリデーション |
|-----------|-----|------|-----------|---------------|
| displayName | string | No | null | 最大50文字 |
| email | string | Yes | - | メールアドレス形式 |
| createdAt | Timestamp | Yes | serverTimestamp() | - |

### アクセスパターン

| パターン | クエリ | 用途 | 頻度 |
|---------|-------|------|------|
| UID指定取得 | `doc(db, 'admins', uid)` | ログイン時の管理者情報取得 | 低 |

### 設計判断

- **ドキュメントID = UID**：Firebase AuthenticationのUIDを使用
- **Custom Claims**：`{ admin: true }` で権限管理（Firestoreではなく認証側）
- **最小限のフィールド**：Phase2では1名運用、拡張性は後回し

---

## /users/{userId}/favorites（お気に入りチェーン）

### 概要

ログインユーザーのお気に入りチェーンを管理。サブコレクションとして設計。

### スキーマ

```typescript
interface Favorite {
  // ドキュメントID = chainId
  chainId: string;               // お気に入り登録したチェーンID

  // メタデータ
  createdAt: Timestamp;          // 登録日時（serverTimestamp()）
}
```

### フィールド詳細

| フィールド | 型 | 必須 | デフォルト | バリデーション |
|-----------|-----|------|-----------|---------------|
| chainId | string | Yes | - | /chains/{chainId}に存在すること |
| createdAt | Timestamp | Yes | serverTimestamp() | - |

### アクセスパターン

| パターン | クエリ | 用途 | 頻度 |
|---------|-------|------|------|
| ユーザーの全お気に入り | `collection(db, 'users', userId, 'favorites')` | お気に入りチェーンID一覧取得 | 高 |
| お気に入り確認 | `doc(db, 'users', userId, 'favorites', chainId)` | 登録状態確認 | 高 |

### 設計判断

- **サブコレクション**：ユーザーに強く紐づく、他ユーザーのお気に入りは参照不要
- **ドキュメントID = chainId**：重複登録防止、存在確認が高速
- **favoriteCount同期**：登録/解除時にTransactionで/chains/{chainId}.favoriteCountを更新

---

## /users（ユーザー情報）※Phase3以降

### 概要

Phase3以降でレビュー機能実装時に使用。Phase2では使用しない。

### スキーマ（予定）

```typescript
interface User {
  // ドキュメントID = Firebase Authentication UID
  id: string;

  // 基本情報
  displayName?: string;          // 表示名
  photoURL?: string;             // プロフィール画像URL

  // メタデータ
  createdAt: Timestamp;          // 作成日時
  updatedAt: Timestamp;          // 更新日時
}
```

---

## /reviews（レビュー）※Phase3以降

### 概要

Phase3以降で実装予定。キャンペーンに対するユーザーレビュー。

### スキーマ（予定）

```typescript
interface Review {
  // ドキュメントID（Firestore自動生成）
  id: string;

  // 関連
  campaignId: string;            // レビュー対象キャンペーンID
  userId: string;                // レビュー投稿者UID

  // レビュー内容
  rating: number;                // 評価（1〜5）
  comment?: string;              // コメント

  // メタデータ
  createdAt: Timestamp;          // 作成日時
  updatedAt: Timestamp;          // 更新日時
}
```

---

## ドキュメント間の関連

```
/chains/{chainId}
    ↑ chainId で参照
    │
/campaigns/{campaignId}
    │
    ↓ フィルタ用に chainId を保持
    │
/users/{userId}/favorites/{chainId}
    │
    ↓ Transaction で favoriteCount を更新
    │
/chains/{chainId}.favoriteCount
```

---

## 見積もりドキュメント数（Phase2）

| コレクション | 見積もり数 | 根拠 |
|-------------|-----------|------|
| /chains | 16件 | 対象チェーン店数 |
| /campaigns | 160件 | 16チェーン × 平均10キャンペーン |
| /admins | 1件 | 1名運用 |
| /users/{userId}/favorites | 500〜1,000件 | 100ユーザー × 平均5〜10件 |

---

## 参考

- [Firestore設計ベストプラクティス](./firestore-best-practices.md)
- [001-admin-panel Firestoreスキーマ](../specs/001-admin-panel/contracts/firestore-schema.md)
- [002-chain-list Firestoreスキーマ](../specs/002-chain-list/contracts/firestore-schema.md)
- [003-favorites Firestoreスキーマ](../specs/003-favorites/contracts/firestore-schema.md)
