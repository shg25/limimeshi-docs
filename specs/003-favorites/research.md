# Research: お気に入り登録（Favorites）

**Feature**: 003-favorites  
**Date**: 2025-11-19  
**Status**: Phase 0 - Research  

## Overview

お気に入り登録機能の実装に必要な技術選定と設計パターンの調査結果

## 1. Firebase Authentication（ログイン状態の管理）

### Decision
Firebase Authentication の `onAuthStateChanged` を使用してログイン状態をリアルタイム監視

### Rationale
- **リアルタイム認証状態監視**: ログイン/ログアウトを即座に検知し、UIを更新
- **型安全**: TypeScript完全対応（User | null）
- **002-menu-listとの一貫性**: 002でも同じパターンを使用
- **複数デバイス対応**: 同じユーザーが複数デバイスからログインした場合でも一貫した状態を維持

### Key APIs
```typescript
import { getAuth, onAuthStateChanged, User } from 'firebase/auth';

const auth = getAuth();

onAuthStateChanged(auth, (user: User | null) => {
  if (user) {
    // ログイン済み → お気に入り登録ボタンを表示
    console.log('Logged in:', user.uid);
  } else {
    // 未ログイン → お気に入り登録ボタンを非表示
    console.log('Not logged in');
  }
});
```

### Alternatives Considered
- **auth.currentUser**: リアルタイム監視できず、ログイン状態の変化を検知できない

### Reference
- [Firebase Authentication](https://firebase.google.com/docs/auth)
- [onAuthStateChanged](https://firebase.google.com/docs/auth/web/manage-users#get_the_currently_signed-in_user)

---

## 2. Firestore（お気に入りデータの永続化）

### Decision
Firestore の `/users/{userId}/favorites/{chainId}` サブコレクションパターンを使用

### Rationale
- **ユーザーごとの隔離**: 各ユーザーのお気に入りデータを独立して管理
- **Security Rulesの簡潔性**: `request.auth.uid == userId` で簡単にアクセス制御
- **スケーラビリティ**: ユーザー数が増えても各ユーザーのお気に入りは独立しているため、クエリパフォーマンスが劣化しない
- **002-menu-listとの一貫性**: 002でも同じパターンで読み取り

### Schema Design

```typescript
// /users/{userId}/favorites/{chainId}
{
  chainId: string;               // お気に入り登録したチェーンID（ドキュメントIDと同じ）
  createdAt: Timestamp;          // 登録日時（Firebase Server Timestamp）
}
```

### Key APIs

```typescript
import { doc, setDoc, deleteDoc, collection, serverTimestamp } from 'firebase/firestore';

// お気に入り登録
const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
await setDoc(favoriteRef, {
  chainId,
  createdAt: serverTimestamp()
});

// お気に入り解除
const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
await deleteDoc(favoriteRef);

// お気に入り状態の確認
const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
const favoriteSnap = await getDoc(favoriteRef);
const isFavorite = favoriteSnap.exists();
```

### Alternatives Considered
- **配列フィールド**: `/users/{userId}` に `favoriteChainIds: string[]` を保存する方法（Security Rulesで要素単位のアクセス制御が難しい、配列サイズ制限）
- **ルートレベルコレクション**: `/favorites/{favoriteId}` にユーザーIDとチェーンIDを保存（クエリが複雑になる）

### Reference
- [Firestore Subcollections](https://firebase.google.com/docs/firestore/data-model#subcollections)

---

## 3. Firestore Transactions（お気に入り登録数の集約データ更新）

### Decision
Firestore Transactions を使用して、お気に入り登録・解除時に `/chains/{chainId}` の `favoriteCount` を増減

### Rationale
- **データ整合性**: お気に入り登録とカウント増加を原子的に実行
- **同時実行制御**: 複数ユーザーが同時にお気に入り登録しても、カウントが正確に増減
- **パフォーマンス**: リアルタイムカウントではなく、集約データとして管理することで、002-menu-listでの表示が高速化

### Key APIs

```typescript
import { runTransaction, doc, increment } from 'firebase/firestore';

// お気に入り登録
await runTransaction(db, async (transaction) => {
  const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
  const chainRef = doc(db, 'chains', chainId);

  // お気に入り登録
  transaction.set(favoriteRef, {
    chainId,
    createdAt: serverTimestamp()
  });

  // お気に入り登録数を+1
  transaction.update(chainRef, {
    favoriteCount: increment(1)
  });
});

// お気に入り解除
await runTransaction(db, async (transaction) => {
  const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
  const chainRef = doc(db, 'chains', chainId);

  // お気に入り解除
  transaction.delete(favoriteRef);

  // お気に入り登録数を-1
  transaction.update(chainRef, {
    favoriteCount: increment(-1)
  });
});
```

### Edge Cases
- **負の値防止**: `favoriteCount` が0の場合、デクリメントしても-1にならないようにする（Firestoreのincrement()は負の値を許容するため、クライアント側で0チェックが必要）
- **トランザクション失敗**: ネットワークエラーや競合でトランザクションが失敗した場合、リトライまたはエラーメッセージを表示

### Alternatives Considered
- **Cloud Functions トリガー**: お気に入り登録時にCloud Functionsで `favoriteCount` を更新する方法（Phase2では不要、コスト増）
- **リアルタイムカウント**: 毎回 `/users/*/favorites/{chainId}` をカウントする方法（パフォーマンス劣化、Firestore読み取り回数増）

### Reference
- [Firestore Transactions](https://firebase.google.com/docs/firestore/manage-data/transactions)

---

## Summary

| 項目 | 選定技術 | 理由 |
|------|---------|------|
| 認証状態管理 | Firebase Authentication (onAuthStateChanged) | リアルタイム監視、型安全性 |
| データ永続化 | Firestore サブコレクション | ユーザーごとの隔離、Security Rules簡潔 |
| 集約データ更新 | Firestore Transactions | データ整合性、同時実行制御 |

全ての技術選定は Constitution に準拠しています。
