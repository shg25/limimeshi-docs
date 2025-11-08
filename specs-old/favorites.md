# 機能設計書：お気に入り登録機能

作成日：2025/10/26

---

## 1. 概要（Overview）

### 機能名

お気に入り登録機能

### 目的

ユーザーが好きなチェーン店をお気に入り登録し、お気に入りチェーンの期間限定メニューのみを一覧表示できるようにする

### 対象ユーザー

- 情報感度の高い中高生（16歳女子）：好きなチェーン（スタバ、マック）のメニューだけ見たい
- 気分転換を求めるOL（28歳女性）：お気に入りのカフェチェーンの新メニューを見逃したくない
- 堅実派会社員（45歳男性）：よく行くチェーン（吉野家、松屋、丸亀製麺）のメニューを優先的に確認したい

### Phase2（MVP）での実装範囲

- チェーン店のお気に入り登録・解除
- お気に入り登録したチェーンのみの一覧表示
- お気に入り状態の永続化（Firestore）
- 匿名認証でもお気に入り登録可能（Googleログインに移行してもUID継続）

### Phase3以降で追加予定

- お気に入りチェーンの新メニュー通知（プッシュ通知、メール通知）
- お気に入りチェーンの並び替え（ドラッグ&ドロップ）
- お気に入りチェーンのグループ化（「カフェ」「ランチ」など）

---

## 2. ユーザーストーリー（User Stories）

### US-1：好きなチェーン店をお気に入り登録したい

**ユーザー**：情報感度の高い中高生（16歳女子）

**ストーリー**：
```
リミメシを初めて使う
メニュー一覧ページで複数のチェーンのメニューが表示されている
「スタバとマックだけ見たい」と思う
各メニューカードの♡ボタンをタップ（またはチェーン詳細ページの♡ボタンをタップ）
お気に入り登録される（♡ → ♥に変化）
「お気に入り」タブをタップ
お気に入り登録したチェーン（スタバ、マック）のメニューのみ表示される
```

**受入基準**：
- メニューカードまたはチェーン詳細ページからお気に入り登録・解除ができる
- お気に入り登録時にボタンの表示が変わる（♡ → ♥）
- お気に入り登録したチェーンのメニューのみ表示できる
- 匿名認証でもお気に入り登録可能

### US-2：お気に入り登録を解除したい

**ユーザー**：気分転換を求めるOL（28歳女性）

**ストーリー**：
```
以前お気に入り登録したチェーン（コメダ珈琲店）があまり行かなくなった
メニュー一覧ページでコメダのメニューカードの♥ボタンをタップ
お気に入り解除される（♥ → ♡に変化）
「お気に入り」タブを見ると、コメダのメニューが表示されなくなった
```

**受入基準**：
- お気に入り登録済みのチェーンを簡単に解除できる
- お気に入り解除時にボタンの表示が変わる（♥ → ♡）
- お気に入り解除後、「お気に入り」タブでそのチェーンのメニューが表示されなくなる

### US-3：匿名認証からGoogleログインに移行してもお気に入りが引き継がれる

**ユーザー**：堅実派会社員（45歳男性）

**ストーリー**：
```
最初は匿名認証でお気に入り登録（吉野家、松屋、丸亀製麺）
後日、Googleログインに切り替える
ログイン後もお気に入りが引き継がれている（吉野家、松屋、丸亀製麺）
他の端末でログインしてもお気に入りが同期されている
```

**受入基準**：
- 匿名認証からGoogleログインに移行してもUIDが継続する
- お気に入り登録は`/users/{uid}/favorites/`に保存される
- ログイン後もお気に入りが引き継がれる
- 他の端末でログインしてもお気に入りが同期される

### US-4：お気に入り未登録の場合は全チェーンのメニューが表示される

**ユーザー**：堅実派会社員（45歳男性）

**ストーリー**：
```
リミメシを初めて使う
まだお気に入り登録していない
メニュー一覧ページで全チェーンのメニューが表示される
「お気に入り」タブをタップしても、全チェーンのメニューが表示される（お気に入り未登録のため）
```

**受入基準**：
- お気に入り未登録の場合は全チェーンのメニューが表示される
- 「お気に入り」タブをタップしてもエラーにならない
- 「お気に入り未登録です。好きなチェーンをお気に入り登録してください。」とメッセージが表示される（任意）

---

## 3. 受け入れ基準（Acceptance Criteria）

### AC-1：お気に入り登録の基本機能

- [ ] メニューカードからお気に入り登録・解除ができる
- [ ] チェーン詳細ページからお気に入り登録・解除ができる
- [ ] お気に入り登録時にボタンの表示が変わる（♡ → ♥）
- [ ] お気に入り解除時にボタンの表示が変わる（♥ → ♡）
- [ ] お気に入り登録・解除はリアルタイムに反映される
- [ ] お気に入り登録には認証が必要（匿名認証でもOK）
- [ ] 未認証の場合はログイン画面に遷移する

### AC-2：お気に入りフィルタリング

- [ ] 「お気に入り」タブをタップすると、お気に入り登録したチェーンのメニューのみ表示される
- [ ] お気に入り未登録の場合は全チェーンのメニューが表示される
- [ ] お気に入り未登録の場合は「お気に入り未登録です」とメッセージが表示される（任意）
- [ ] お気に入り登録したチェーンが多い場合でも正しく動作する（最大10チェーンまで、Firestoreの`in`クエリ制限）

### AC-3：お気に入り状態の永続化

- [ ] お気に入り登録はFirestoreに保存される（`/users/{uid}/favorites/{favoriteId}`）
- [ ] ページをリロードしてもお気に入り状態が保持される
- [ ] 匿名認証からGoogleログインに移行してもお気に入りが引き継がれる（UID継続）
- [ ] 他の端末でログインしてもお気に入りが同期される

### AC-4：エラーハンドリング

- [ ] お気に入り登録エラー時は適切なエラーメッセージが表示される
- [ ] お気に入り解除エラー時は適切なエラーメッセージが表示される
- [ ] ネットワークエラー時はリトライボタンが表示される
- [ ] 認証エラー時はログイン画面に遷移する

### AC-5：パフォーマンス

- [ ] お気に入り登録・解除は1秒以内に完了
- [ ] お気に入り状態の取得は500ms以内
- [ ] お気に入り登録したチェーンが多い場合でも正しく動作する（最大10チェーンまで）

### AC-6：アクセシビリティ

- [ ] お気に入りボタンにaria-labelを設定（「お気に入り登録」「お気に入り解除」）
- [ ] キーボード操作が可能（Tab、Enterキー）
- [ ] スクリーンリーダーでの読み上げに対応

---

## 4. 技術仕様（Technical Specification）

### 4-1. データモデル

#### Favorites（お気に入り）

```typescript
interface Favorite {
  id: string; // お気に入りID（自動生成）
  userId: string; // ユーザーID（外部キー、UIDと同じ）
  chainId: string; // チェーンID（外部キー）
  createdAt: Date; // 登録日時
}
```

### 4-2. Firestoreコレクション設計

```
/users/{userId}/favorites/{favoriteId}
```

- `userId`：Firebase Authentication の UID
- `favoriteId`：自動生成（Firestore の auto ID）
- サブコレクションとして `favorites` を配置（ユーザーごとに隔離）

### 4-3. Firestoreクエリ

#### お気に入り一覧取得

```typescript
// 特定ユーザーのお気に入りチェーンIDを取得
const favoritesQuery = query(
  collection(db, `users/${userId}/favorites`)
);
const favoritesSnapshot = await getDocs(favoritesQuery);
const favoriteChainIds = favoritesSnapshot.docs.map(doc => doc.data().chainId);

// お気に入りチェーンの期間限定メニューを取得
const menusQuery = query(
  collection(db, 'menus'),
  where('chainId', 'in', favoriteChainIds), // 最大10件まで
  where('menuType', '==', 'limited'),
  where('status', '==', 'on_sale'),
  orderBy('updatedAt', 'desc'),
  limit(20)
);
```

#### お気に入り登録

```typescript
// お気に入り登録
const favoriteRef = collection(db, `users/${userId}/favorites`);
await addDoc(favoriteRef, {
  chainId: chainId,
  createdAt: serverTimestamp()
});
```

#### お気に入り解除

```typescript
// お気に入り解除（chainIdで検索して削除）
const favoritesQuery = query(
  collection(db, `users/${userId}/favorites`),
  where('chainId', '==', chainId)
);
const favoritesSnapshot = await getDocs(favoritesQuery);
favoritesSnapshot.docs.forEach(async (doc) => {
  await deleteDoc(doc.ref);
});
```

#### お気に入り状態確認

```typescript
// 特定チェーンがお気に入り登録されているか確認
const favoritesQuery = query(
  collection(db, `users/${userId}/favorites`),
  where('chainId', '==', chainId)
);
const favoritesSnapshot = await getDocs(favoritesQuery);
const isFavorite = !favoritesSnapshot.empty;
```

### 4-4. インデックス設計

Phase1-4（データモデル詳細設計）で詳細化予定

必要なインデックス（概要）：
- `favorites`サブコレクション：`chainId`（お気に入り解除、状態確認で使用）

### 4-5. セキュリティルール

Phase1-4（データモデル詳細設計）で詳細化予定

基本方針：
- お気に入りは自分のデータのみ読み書き可能
- `userId`がFirebase Authentication の UID と一致することを確認

```javascript
// Security Rules（概要）
match /users/{userId}/favorites/{favoriteId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

### 4-6. コンポーネント設計

```
components/
├── FavoriteButton.tsx             # お気に入りボタン（♡ / ♥）
├── FavoriteTab.tsx                # お気に入りタブ
└── FavoriteEmptyState.tsx         # お気に入り未登録時の表示

hooks/
├── useFavorites.ts                # お気に入り管理フック
└── useAuth.ts                     # 認証管理フック
```

### 4-7. 状態管理

- React Context API または Zustand を使用
- お気に入り状態（Firestore）
- 認証状態（Firebase Authentication）

```typescript
// useFavorites フック（概要）
interface UseFavoritesReturn {
  favoriteChainIds: string[]; // お気に入りチェーンIDの配列
  isFavorite: (chainId: string) => boolean; // 特定チェーンがお気に入りか確認
  addFavorite: (chainId: string) => Promise<void>; // お気に入り登録
  removeFavorite: (chainId: string) => Promise<void>; // お気に入り解除
  isLoading: boolean; // ローディング状態
  error: Error | null; // エラー状態
}
```

### 4-8. 匿名認証からGoogleログインへの移行

```typescript
// 匿名認証からGoogleログインへの移行（UID継続）
const auth = getAuth();
const currentUser = auth.currentUser;

if (currentUser && currentUser.isAnonymous) {
  const provider = new GoogleAuthProvider();
  try {
    // linkWithPopup でUIDを継続
    const credential = await linkWithPopup(currentUser, provider);
    console.log('匿名ユーザーをGoogleアカウントにリンクしました', credential.user);
  } catch (error) {
    console.error('リンクエラー', error);
  }
}
```

---

## 5. UI/UX設計（UI/UX Design）

### 5-1. 画面構成

#### メニュー一覧ページ（お気に入りボタン表示）

```
+----------------------------------+
| リミメシ                    [☰] |
+----------------------------------+
| [全て] [お気に入り] [チェーン▼] |
+----------------------------------+
| +------------------------------+ |
| | [画像]                       | |
| | 月見バーガー               [♥]| | ← お気に入り登録済み（♥）
| | マクドナルド                 | |
| | ¥390 | 9/1〜10/31            | |
| +------------------------------+ |
| +------------------------------+ |
| | [画像]                       | |
| | チゲ味噌ラーメン           [♡]| | ← お気に入り未登録（♡）
| | 日高屋                       | |
| | ¥690 | 期間未定              | |
| +------------------------------+ |
| ...                              |
+----------------------------------+
```

#### チェーン詳細ページ（お気に入りボタン表示）

```
+----------------------------------+
| [←] マクドナルド          [♥]  | ← お気に入り登録済み（♥）
+----------------------------------+
| [ロゴ画像]                       |
| マクドナルド                     |
| タグ：ハンバーガー               |
| [公式サイトを見る]               |
+----------------------------------+
| 期間限定メニュー                 |
+----------------------------------+
| ...                              |
+----------------------------------+
```

#### お気に入り未登録時の表示

```
+----------------------------------+
| リミメシ                    [☰] |
+----------------------------------+
| [全て] [お気に入り] [チェーン▼] |
+----------------------------------+
|                                  |
|        [♡アイコン]               |
|                                  |
|   お気に入り未登録です           |
|                                  |
| 好きなチェーンをお気に入り       |
| 登録してください                 |
|                                  |
|   [全てのメニューを見る]         |
|                                  |
+----------------------------------+
```

### 5-2. インタラクション

#### お気に入りボタン

- タップでお気に入り登録・解除
- お気に入り未登録時は♡（白抜き）、登録時は♥（塗りつぶし）
- タップ時にアニメーション（拡大縮小、色変化）
- タップ時にバイブレーション（モバイルのみ、任意）

#### お気に入りタブ

- 「お気に入り」タブをタップすると、お気に入り登録したチェーンのメニューのみ表示
- お気に入り未登録の場合は、お気に入り未登録画面を表示
- 「全てのメニューを見る」ボタンをタップすると、全チェーンのメニューが表示される

#### 認証フロー

- 未認証の場合はお気に入りボタンをタップすると匿名認証が実行される
- 匿名認証成功後、お気に入り登録が実行される
- 匿名認証エラー時は適切なエラーメッセージが表示される

### 5-3. デザイン方針

- お気に入りボタンは視認性の高いデザイン（♡ / ♥）
- お気に入り登録時は視覚的フィードバック（アニメーション、色変化）
- お気に入り未登録時は分かりやすいメッセージとCTA（「全てのメニューを見る」ボタン）

---

## 6. テスト計画（Test Plan）

### 6-1. 単体テスト

#### 対象

- useFavorites フック
- useAuth フック
- お気に入り登録・解除ロジック

#### ツール

- Jest、Vitest
- Firebase Emulator（Firestore、Auth）

#### テストケース例

```typescript
describe('useFavorites', () => {
  it('お気に入り登録ができる', async () => {
    const { result } = renderHook(() => useFavorites());
    await act(async () => {
      await result.current.addFavorite('chain-id-1');
    });
    expect(result.current.favoriteChainIds).toContain('chain-id-1');
  });

  it('お気に入り解除ができる', async () => {
    const { result } = renderHook(() => useFavorites());
    await act(async () => {
      await result.current.addFavorite('chain-id-1');
      await result.current.removeFavorite('chain-id-1');
    });
    expect(result.current.favoriteChainIds).not.toContain('chain-id-1');
  });

  it('お気に入り状態が正しく取得できる', async () => {
    const { result } = renderHook(() => useFavorites());
    await act(async () => {
      await result.current.addFavorite('chain-id-1');
    });
    expect(result.current.isFavorite('chain-id-1')).toBe(true);
    expect(result.current.isFavorite('chain-id-2')).toBe(false);
  });
});
```

### 6-2. E2Eテスト

#### 対象

- お気に入り登録・解除
- お気に入りフィルタリング
- 匿名認証からGoogleログインへの移行

#### ツール

- Playwright、Cypress
- Firebase Emulator（Firestore、Auth）

#### テストシナリオ例

```typescript
test('お気に入り登録・解除ができる', async ({ page }) => {
  await page.goto('/');

  // お気に入り登録
  await page.click('.favorite-button:first-child');
  await expect(page.locator('.favorite-button:first-child')).toHaveClass(/active/);

  // お気に入り解除
  await page.click('.favorite-button:first-child');
  await expect(page.locator('.favorite-button:first-child')).not.toHaveClass(/active/);
});

test('お気に入りフィルタリングができる', async ({ page }) => {
  await page.goto('/');

  // お気に入り登録
  await page.click('.favorite-button:first-child');

  // お気に入りタブをタップ
  await page.click('text=お気に入り');

  // お気に入り登録したチェーンのメニューのみ表示される
  await expect(page.locator('.menu-card')).toHaveCount(1);
});
```

### 6-3. 負荷テスト

#### 対象

- お気に入り登録・解除のパフォーマンス
- お気に入り一覧取得のパフォーマンス

#### ツール

- Artillery、k6、Firebase Emulator

#### テストシナリオ

- お気に入り登録・解除：100回/秒
- 目標レスポンス時間：1秒以内（p95）

### 6-4. アクセシビリティテスト

#### 対象

- お気に入りボタンのaria-label
- キーボード操作
- スクリーンリーダー対応

#### ツール

- axe DevTools
- Lighthouse

#### 確認項目

- [ ] お気に入りボタンにaria-labelを設定（「お気に入り登録」「お気に入り解除」）
- [ ] キーボード操作が可能（Tab、Enterキー）
- [ ] スクリーンリーダーでの読み上げに対応

### 6-5. テストカバレッジ目標

- 単体テスト：70%以上
- E2Eテスト：お気に入り登録・解除、フィルタリング
- 負荷テスト：Phase2リリース前必須
- アクセシビリティテスト：Lighthouse スコア90以上

---

## 7. 非機能要件（Non-Functional Requirements）

### 7-1. パフォーマンス

- お気に入り登録・解除：1秒以内
- お気に入り状態の取得：500ms以内
- お気に入り一覧取得：1秒以内

### 7-2. スケーラビリティ

- お気に入り登録数：ユーザーあたり最大10チェーンまで（Firestoreの`in`クエリ制限）
- お気に入り登録数が10を超える場合は、複数回クエリを実行（Phase3以降で検討）

### 7-3. セキュリティ

- お気に入りは自分のデータのみ読み書き可能
- Security Rulesで`userId`がFirebase Authentication の UID と一致することを確認

### 7-4. 信頼性

- お気に入り登録・解除エラー時は適切なエラーメッセージが表示される
- ネットワークエラー時はリトライボタンが表示される
- 認証エラー時はログイン画面に遷移する

### 7-5. 保守性

- TypeScriptで型安全性を確保
- useFavorites フックで状態管理を一元化
- 単体テスト・E2Eテストを実装と同時に作成

---

## 8. 残課題（TBD）

### Phase1で検討

- [ ] お気に入り登録数の上限設定（10チェーンまで、Phase2では制限なし）
- [ ] Security Rulesの詳細化（data-model/security-rules.md）
- [ ] お気に入り登録時のアニメーション詳細

### Phase2で実装

- [ ] お気に入りボタンのデザイン最終化
- [ ] お気に入り未登録時のメッセージ文言最終化
- [ ] 匿名認証からGoogleログインへの移行フローのテスト
- [ ] 負荷テストの実施とパフォーマンスチューニング

### Phase3以降で追加

- [ ] お気に入りチェーンの新メニュー通知（プッシュ通知、メール通知）
- [ ] お気に入りチェーンの並び替え（ドラッグ&ドロップ）
- [ ] お気に入りチェーンのグループ化（「カフェ」「ランチ」など）
- [ ] お気に入り登録数が10を超える場合の対応（複数回クエリ実行）

---

## 9. 参考資料

- [planning/first-idea.md](../planning/first-idea.md)：サービス全体像、ペルソナ、データモデル
- [planning/lean-canvas.md](../planning/lean-canvas.md)：ビジネスモデル、ターゲットユーザー
- [planning/inception-deck.md](../planning/inception-deck.md)：ユーザージャーニー、優先順位
- [specs/menu-list.md](./menu-list.md)：メニュー一覧機能（お気に入りフィルタリングと連携）
- [WRITING_STYLE_GUIDE.md](../WRITING_STYLE_GUIDE.md)：ドキュメント記述ルール

---

## 更新履歴

- 2025/10/26：初版作成
