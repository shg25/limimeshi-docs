# Quickstart: お気に入り登録（Favorites）

**Feature**: 003-favorites  
**Date**: 2025-11-19  
**Related**: [spec.md](./spec.md), [research.md](./research.md), [contracts/firestore-schema.md](./contracts/firestore-schema.md)  

## Prerequisites

- Node.js 18+ インストール済み
- Firebase Project作成済み（`limimeshi-dev` or `limimeshi-prod`）
- Firestore有効化済み
- Firebase Authentication有効化済み（Googleログイン、匿名認証）
- limimeshi-web リポジトリがセットアップ済み
- 002-menu-list機能が実装済み（お気に入りフィルタとの連携のため）

---

## Core Implementation

### 1. お気に入りボタンコンポーネント

`src/components/FavoriteButton.tsx`:
```typescript
import { useState, useEffect } from 'react';
import { IconButton, CircularProgress } from '@mui/material';
import { Favorite, FavoriteBorder } from '@mui/icons-material';
import { doc, getDoc, runTransaction, serverTimestamp, increment } from 'firebase/firestore';
import { db, auth } from '../firebase/config';
import { User } from 'firebase/auth';

interface FavoriteButtonProps {
  chainId: string;
  user: User | null;
}

export const FavoriteButton: React.FC<FavoriteButtonProps> = ({ chainId, user }) => {
  const [isFavorite, setIsFavorite] = useState<boolean>(false);
  const [loading, setLoading] = useState<boolean>(false);

  useEffect(() => {
    const checkFavoriteStatus = async () => {
      if (!user) {
        setIsFavorite(false);
        return;
      }

      const favoriteRef = doc(db, 'users', user.uid, 'favorites', chainId);
      const favoriteSnap = await getDoc(favoriteRef);
      setIsFavorite(favoriteSnap.exists());
    };

    checkFavoriteStatus();
  }, [user, chainId]);

  const handleToggleFavorite = async () => {
    if (!user) return;

    setLoading(true);
    try {
      await runTransaction(db, async (transaction) => {
        const favoriteRef = doc(db, 'users', user.uid, 'favorites', chainId);
        const chainRef = doc(db, 'chains', chainId);

        if (isFavorite) {
          transaction.delete(favoriteRef);
          transaction.update(chainRef, {
            favoriteCount: increment(-1),
            updatedAt: serverTimestamp()
          });
        } else {
          transaction.set(favoriteRef, {
            chainId,
            createdAt: serverTimestamp()
          });
          transaction.update(chainRef, {
            favoriteCount: increment(1),
            updatedAt: serverTimestamp()
          });
        }
      });

      setIsFavorite(!isFavorite);
    } catch (err) {
      console.error('Error toggling favorite:', err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <IconButton
      onClick={handleToggleFavorite}
      disabled={!user || loading}
      aria-label={isFavorite ? 'お気に入り解除' : 'お気に入り登録'}
    >
      {loading ? <CircularProgress size={24} /> : isFavorite ? <Favorite color="error" /> : <FavoriteBorder />}
    </IconButton>
  );
};
```

---

## Testing

### 単体テスト

`tests/unit/FavoriteButton.test.tsx`:
```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { FavoriteButton } from '../../src/components/FavoriteButton';

describe('FavoriteButton', () => {
  it('ログインユーザーの場合、お気に入り登録ボタンが表示される', () => {
    const user = { uid: 'test-user' } as any;
    render(<FavoriteButton chainId="chain123" user={user} />);
    expect(screen.getByLabelText('お気に入り登録')).toBeInTheDocument();
  });

  it('未ログインユーザーの場合、ボタンが無効化される', () => {
    render(<FavoriteButton chainId="chain123" user={null} />);
    const button = screen.getByLabelText('お気に入り登録');
    expect(button).toBeDisabled();
  });
});
```
