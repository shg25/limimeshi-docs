# Firestore Schema: お気に入り登録（Favorites）

**Feature**: 003-favorites  
**Date**: 2025-11-19  
**Status**: Phase 1 - Contract  
**Related**: [research.md](../research.md), [002-menu-list Firestore Schema](../../002-menu-list/contracts/firestore-schema.md)

## Overview

お気に入り登録機能で使用するFirestoreコレクション構造（読み書き可能）

## Collections

### /users/{userId}/favorites/{chainId}

ユーザーごとのお気に入りチェーン店（003-favoritesで登録、002-menu-listで読み取り）

```typescript
{
  // Firestore自動生成ID（ドキュメントID） = chainId
  chainId: string;               // お気に入り登録したチェーンID

  // メタデータ
  createdAt: Timestamp;          // 登録日時（Firebase Server Timestamp）
}
```

**Operations Used**:

1. **お気に入り登録**:
   ```typescript
   import { doc, setDoc, serverTimestamp, runTransaction, increment } from 'firebase/firestore';

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
       favoriteCount: increment(1),
       updatedAt: serverTimestamp()
     });
   });
   ```

2. **お気に入り解除**:
   ```typescript
   await runTransaction(db, async (transaction) => {
     const favoriteRef = doc(db, 'users', userId, 'favorites', chainId);
     const chainRef = doc(db, 'chains', chainId);

     // お気に入り解除
     transaction.delete(favoriteRef);

     // お気に入り登録数を-1
     transaction.update(chainRef, {
       favoriteCount: increment(-1),
       updatedAt: serverTimestamp()
     });
   });
   ```

---

## Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // お気に入りサブコレクション
    match /users/{userId}/favorites/{chainId} {
      // ログインユーザー本人のみ読み書き可能
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // チェーンコレクション（お気に入り登録数の更新）
    match /chains/{chainId} {
      // 全ユーザーが読み取り可能（001, 002で使用）
      allow read: if true;

      // お気に入り登録・解除時のfavoriteCount更新のみ許可
      allow update: if request.auth != null
                    && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['favoriteCount', 'updatedAt'])
                    && request.resource.data.favoriteCount >= 0;  // 負の値を防止
    }
  }
}
```

---

## Notes

- **読み書き可能**: 003-favoritesはお気に入りデータの読み書きを行う
- **002との連携**: 002-menu-listはお気に入りデータを読み取り専用で使用
- **001との整合性**: /chainsのスキーマは001-admin-panelと完全一致（favoriteCountフィールドを追加）
- **Transaction使用**: お気に入り登録・解除時はTransactionを使用してデータ整合性を保証
