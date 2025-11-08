# 機能設計書：管理画面機能

作成日：2025/10/26

---

## 1. 概要（Overview）

### 機能名

管理画面機能

### 目的

管理者がチェーン店、メニュー、タグを手動で登録・編集・削除できる管理画面を提供する

### 対象ユーザー

- 管理者（重次弘規、将来的には協力者も想定）

### Phase2（MVP）での実装範囲

- チェーン店管理（登録・編集・削除・一覧表示）
- メニュー管理（登録・編集・削除・一覧表示）
- タグ管理（登録・編集・削除・一覧表示）
- 入力補助機能（コピペしやすいフォーム設計、過去の類似メニューからの入力候補表示）
- 管理者認証（メール + パスワード、Custom Claims）

### Phase3以降で追加予定

- ユーザー管理（一般ユーザーの一覧表示、削除、統計情報）
- レビュー管理（一覧表示、削除、モデレーション）
- 販売終了メニューの一括更新機能
- メニュー登録の効率化（公式サイトのURLを入力すると情報を抽出する機能、Phase3以降）
- ダッシュボード（KPI、アクセス解析、コスト監視）

---

## 2. ユーザーストーリー（User Stories）

### US-1：新しいチェーン店を登録したい

**ユーザー**：管理者

**ストーリー**：
```
管理画面にログイン（メール + パスワード、Custom Claims）
チェーン店管理ページを開く
「新規作成」ボタンをクリック
チェーン名、タグ、公式サイトURL、ロゴ画像URLを入力
「保存」ボタンをクリック
チェーン店が登録される
チェーン店一覧ページに新しいチェーン店が表示される
```

**受入基準**：
- チェーン店の新規登録ができる
- チェーン名は必須、タグ・公式サイトURL・ロゴ画像URLは任意
- タグは複数選択可能
- 保存後、チェーン店一覧ページに遷移する

### US-2：新しいメニューを登録したい

**ユーザー**：管理者

**ストーリー**：
```
メニュー管理ページを開く
「新規作成」ボタンをクリック
所属チェーンを選択（ドロップダウン）
メニュー名、説明、価格、画像URL、メニュータイプ（期間限定 / 通常）、販売期間、ステータスを入力
入力補助機能が働き、過去の類似メニューから入力候補が表示される
「保存」ボタンをクリック
メニューが登録される
メニュー一覧ページに新しいメニューが表示される
```

**受入基準**：
- メニューの新規登録ができる
- メニュー名と所属チェーンは必須、その他は任意
- 入力補助機能が働き、過去の類似メニューから入力候補が表示される
- 販売期間の終了日は「未定」を選択可能
- 保存後、メニュー一覧ページに遷移する

### US-3：メニューを編集したい

**ユーザー**：管理者

**ストーリー**：
```
メニュー一覧ページで編集したいメニューを検索
「編集」ボタンをクリック
メニュー情報を修正（販売期間の終了日を更新、ステータスを「販売終了」に変更など）
「保存」ボタンをクリック
メニューが更新される
一般向けWebアプリに反映される
```

**受入基準**：
- メニューの編集ができる
- 編集後、Firestoreに保存される
- 一般向けWebアプリにリアルタイムで反映される

### US-4：販売終了したメニューを一括更新したい

**ユーザー**：管理者（Phase3以降）

**ストーリー**：
```
メニュー一覧ページで販売終了したメニューを複数選択
「一括更新」ボタンをクリック
ステータスを「販売終了」に一括変更
「保存」ボタンをクリック
複数のメニューが一括更新される
```

**受入基準**：
- メニューの一括更新ができる（Phase3以降）
- 複数選択可能
- ステータスを「販売終了」に一括変更できる

### US-5：タグを管理したい

**ユーザー**：管理者

**ストーリー**：
```
タグ管理ページを開く
「新規作成」ボタンをクリック
タグ名、表示順を入力
「保存」ボタンをクリック
タグが登録される
チェーン店登録時にタグを選択できる
```

**受入基準**：
- タグの新規登録・編集・削除ができる
- タグ名は必須、表示順は任意
- タグはチェーン店登録時に選択できる

---

## 3. 受け入れ基準（Acceptance Criteria）

### AC-1：管理者認証

- [ ] 管理画面はメール + パスワードでログイン
- [ ] Custom Claimsで`admin: true`を設定
- [ ] 一般ユーザーは管理画面にアクセスできない（Security Rulesで制御）
- [ ] ログアウト機能がある

### AC-2：チェーン店管理

- [ ] チェーン店の新規登録・編集・削除ができる
- [ ] チェーン店一覧が表示される
- [ ] チェーン名で検索・フィルタリングができる
- [ ] タグで検索・フィルタリングができる
- [ ] チェーン名は必須、タグ・公式サイトURL・ロゴ画像URLは任意
- [ ] タグは複数選択可能

### AC-3：メニュー管理

- [ ] メニューの新規登録・編集・削除ができる
- [ ] メニュー一覧が表示される
- [ ] メニュー名、チェーン名、メニュータイプ、ステータスで検索・フィルタリングができる
- [ ] メニュー名と所属チェーンは必須、その他は任意
- [ ] 販売期間の終了日は「未定」を選択可能
- [ ] 入力補助機能が働き、過去の類似メニューから入力候補が表示される

### AC-4：タグ管理

- [ ] タグの新規登録・編集・削除ができる
- [ ] タグ一覧が表示される
- [ ] タグ名は必須、表示順は任意
- [ ] タグはチェーン店登録時に選択できる

### AC-5：入力補助機能

- [ ] メニュー登録時に、過去の類似メニューから入力候補が表示される
- [ ] チェーン店を選択すると、そのチェーンの過去のメニューが表示される
- [ ] メニュー名を入力すると、類似するメニューが表示される
- [ ] コピペしやすいフォーム設計（URLを貼り付けるとブラウザで開くボタンが表示される、など）

### AC-6：エラーハンドリング

- [ ] 保存エラー時は適切なエラーメッセージが表示される
- [ ] 削除確認ダイアログが表示される（誤操作防止）
- [ ] 必須項目が未入力の場合は保存できない

### AC-7：パフォーマンス

- [ ] ページ読み込み時間が3秒以内
- [ ] 保存・削除操作が1秒以内に完了
- [ ] 一覧ページはページネーション実装（1ページあたり50件程度）

### AC-8：アクセシビリティ

- [ ] セマンティックHTMLを使用
- [ ] キーボード操作が可能
- [ ] カラーコントラストがWCAG2.1 AA準拠

---

## 4. 技術仕様（Technical Specification）

### 4-1. データモデル

#### Admins（管理者）

```typescript
interface Admin {
  id: string; // 管理者ID（Firebase UID）
  displayName: string; // 表示名
  email: string; // メールアドレス
  authMethod: 'email'; // 認証方法（メール + パスワード）
  customClaims: {
    admin: true; // Custom Claims
  };
  createdAt: Date; // 作成日時
  lastLoginAt: Date; // 最終ログイン日時
}
```

#### Chains（チェーン店）

```typescript
interface Chain {
  id: string; // チェーンID（自動生成）
  name: string; // チェーン名（必須）
  tagIds: string[]; // タグID（配列、複数タグ対応）
  officialUrl?: string; // 公式サイトURL（任意）
  logoUrl?: string; // ロゴ画像URL（任意）
  createdAt: Date; // 作成日時
  updatedAt: Date; // 更新日時
}
```

#### Menus（メニュー）

```typescript
interface Menu {
  id: string; // メニューID（自動生成）
  chainId: string; // 所属チェーンID（外部キー、必須）
  name: string; // メニュー名（必須）
  description?: string; // 説明（任意）
  price?: number; // 価格（任意、円単位）
  imageUrl?: string; // 画像URL（任意）
  imageType: 'template' | 'official'; // 画像タイプ（テンプレート / 公式）
  menuType: 'limited' | 'regular'; // メニュータイプ（期間限定 / 通常）
  salesPeriod?: {
    startDate?: Date; // 開始日（任意）
    endDate?: Date; // 終了日（任意、未定の場合はnull）
  };
  status: 'on_sale' | 'ended' | 'scheduled'; // ステータス（販売中 / 販売終了 / 予定）
  createdAt: Date; // 作成日時
  updatedAt: Date; // 更新日時
}
```

#### Tags（タグ）

```typescript
interface Tag {
  id: string; // タグID（自動生成）
  name: string; // タグ名（必須、例：「ハンバーガー」「カフェ」「丼」）
  displayOrder: number; // 表示順（任意）
  createdAt: Date; // 作成日時
}
```

### 4-2. Firestoreコレクション設計

```
/admins/{adminId}
/chains/{chainId}
/menus/{menuId}
/tags/{tagId}
```

### 4-3. 認証・権限管理

#### 管理者認証

- Firebase Authenticationでメール + パスワード認証
- Custom Claimsで`admin: true`を設定
- Firestoreは`/admins/{uid}`コレクションで管理（一般ユーザーと分離）

#### Custom Claims設定

```typescript
// Cloud Functions（管理者作成時に実行）
const admin = require('firebase-admin');

async function setAdminClaim(uid: string) {
  await admin.auth().setCustomUserClaims(uid, { admin: true });
}
```

#### Security Rules

```javascript
// Security Rules（概要）
function isAdmin() {
  return request.auth != null && request.auth.token.admin == true;
}

match /chains/{chainId} {
  allow read: if true; // 全ユーザーが読み取り可能
  allow write: if isAdmin(); // 管理者のみ書き込み可能
}

match /menus/{menuId} {
  allow read: if true; // 全ユーザーが読み取り可能
  allow write: if isAdmin(); // 管理者のみ書き込み可能
}

match /tags/{tagId} {
  allow read: if true; // 全ユーザーが読み取り可能
  allow write: if isAdmin(); // 管理者のみ書き込み可能
}

match /admins/{adminId} {
  allow read: if isAdmin() && request.auth.uid == adminId; // 自分のデータのみ読み取り可能
  allow write: if false; // 書き込み不可（Cloud Functionsから設定）
}
```

### 4-4. React Admin設定

#### 基本構成

```typescript
// src/admin/App.tsx
import { Admin, Resource } from 'react-admin';
import { FirebaseAuthProvider, FirebaseDataProvider } from 'react-admin-firebase';
import { ChainList, ChainEdit, ChainCreate } from './chains';
import { MenuList, MenuEdit, MenuCreate } from './menus';
import { TagList, TagEdit, TagCreate } from './tags';

const firebaseConfig = {
  // Firebase設定
};

const authProvider = FirebaseAuthProvider(firebaseConfig);
const dataProvider = FirebaseDataProvider(firebaseConfig);

function App() {
  return (
    <Admin authProvider={authProvider} dataProvider={dataProvider}>
      <Resource name="chains" list={ChainList} edit={ChainEdit} create={ChainCreate} />
      <Resource name="menus" list={MenuList} edit={MenuEdit} create={MenuCreate} />
      <Resource name="tags" list={TagList} edit={TagEdit} create={TagCreate} />
    </Admin>
  );
}

export default App;
```

#### チェーン店管理

```typescript
// src/admin/chains.tsx
import { List, Datagrid, TextField, EditButton, DeleteButton } from 'react-admin';
import { Edit, SimpleForm, TextInput, ReferenceArrayInput, SelectArrayInput } from 'react-admin';
import { Create } from 'react-admin';

export const ChainList = () => (
  <List>
    <Datagrid>
      <TextField source="name" label="チェーン名" />
      <TextField source="officialUrl" label="公式サイトURL" />
      <EditButton />
      <DeleteButton />
    </Datagrid>
  </List>
);

export const ChainEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="name" label="チェーン名" required />
      <ReferenceArrayInput source="tagIds" reference="tags" label="タグ">
        <SelectArrayInput optionText="name" />
      </ReferenceArrayInput>
      <TextInput source="officialUrl" label="公式サイトURL" />
      <TextInput source="logoUrl" label="ロゴ画像URL" />
    </SimpleForm>
  </Edit>
);

export const ChainCreate = () => (
  <Create>
    <SimpleForm>
      <TextInput source="name" label="チェーン名" required />
      <ReferenceArrayInput source="tagIds" reference="tags" label="タグ">
        <SelectArrayInput optionText="name" />
      </ReferenceArrayInput>
      <TextInput source="officialUrl" label="公式サイトURL" />
      <TextInput source="logoUrl" label="ロゴ画像URL" />
    </SimpleForm>
  </Create>
);
```

#### メニュー管理

```typescript
// src/admin/menus.tsx
import { List, Datagrid, TextField, ReferenceField, EditButton, DeleteButton } from 'react-admin';
import { Edit, SimpleForm, TextInput, ReferenceInput, SelectInput, NumberInput, DateInput } from 'react-admin';
import { Create } from 'react-admin';

export const MenuList = () => (
  <List>
    <Datagrid>
      <TextField source="name" label="メニュー名" />
      <ReferenceField source="chainId" reference="chains" label="チェーン名">
        <TextField source="name" />
      </ReferenceField>
      <TextField source="menuType" label="メニュータイプ" />
      <TextField source="status" label="ステータス" />
      <EditButton />
      <DeleteButton />
    </Datagrid>
  </List>
);

export const MenuEdit = () => (
  <Edit>
    <SimpleForm>
      <ReferenceInput source="chainId" reference="chains" label="所属チェーン" required>
        <SelectInput optionText="name" />
      </ReferenceInput>
      <TextInput source="name" label="メニュー名" required />
      <TextInput source="description" label="説明" multiline />
      <NumberInput source="price" label="価格（円）" />
      <TextInput source="imageUrl" label="画像URL" />
      <SelectInput source="imageType" label="画像タイプ" choices={[
        { id: 'template', name: 'テンプレート' },
        { id: 'official', name: '公式' },
      ]} defaultValue="template" />
      <SelectInput source="menuType" label="メニュータイプ" choices={[
        { id: 'limited', name: '期間限定' },
        { id: 'regular', name: '通常' },
      ]} defaultValue="limited" />
      <DateInput source="salesPeriod.startDate" label="販売開始日" />
      <DateInput source="salesPeriod.endDate" label="販売終了日" />
      <SelectInput source="status" label="ステータス" choices={[
        { id: 'on_sale', name: '販売中' },
        { id: 'ended', name: '販売終了' },
        { id: 'scheduled', name: '予定' },
      ]} defaultValue="on_sale" />
    </SimpleForm>
  </Edit>
);

export const MenuCreate = () => (
  <Create>
    <SimpleForm>
      <ReferenceInput source="chainId" reference="chains" label="所属チェーン" required>
        <SelectInput optionText="name" />
      </ReferenceInput>
      <TextInput source="name" label="メニュー名" required />
      <TextInput source="description" label="説明" multiline />
      <NumberInput source="price" label="価格（円）" />
      <TextInput source="imageUrl" label="画像URL" />
      <SelectInput source="imageType" label="画像タイプ" choices={[
        { id: 'template', name: 'テンプレート' },
        { id: 'official', name: '公式' },
      ]} defaultValue="template" />
      <SelectInput source="menuType" label="メニュータイプ" choices={[
        { id: 'limited', name: '期間限定' },
        { id: 'regular', name: '通常' },
      ]} defaultValue="limited" />
      <DateInput source="salesPeriod.startDate" label="販売開始日" />
      <DateInput source="salesPeriod.endDate" label="販売終了日" />
      <SelectInput source="status" label="ステータス" choices={[
        { id: 'on_sale', name: '販売中' },
        { id: 'ended', name: '販売終了' },
        { id: 'scheduled', name: '予定' },
      ]} defaultValue="on_sale" />
    </SimpleForm>
  </Create>
);
```

#### タグ管理

```typescript
// src/admin/tags.tsx
import { List, Datagrid, TextField, NumberField, EditButton, DeleteButton } from 'react-admin';
import { Edit, SimpleForm, TextInput, NumberInput } from 'react-admin';
import { Create } from 'react-admin';

export const TagList = () => (
  <List>
    <Datagrid>
      <TextField source="name" label="タグ名" />
      <NumberField source="displayOrder" label="表示順" />
      <EditButton />
      <DeleteButton />
    </Datagrid>
  </List>
);

export const TagEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="name" label="タグ名" required />
      <NumberInput source="displayOrder" label="表示順" />
    </SimpleForm>
  </Edit>
);

export const TagCreate = () => (
  <Create>
    <SimpleForm>
      <TextInput source="name" label="タグ名" required />
      <NumberInput source="displayOrder" label="表示順" />
    </SimpleForm>
  </Create>
);
```

### 4-5. デプロイ

- Firebase Hosting（`limimeshi-admin`）
- React Adminアプリをビルドしてデプロイ
- URLは `https://admin.limimeshi.com` または `https://limimeshi-admin.web.app`

---

## 5. UI/UX設計（UI/UX Design）

### 5-1. 画面構成

React Adminのデフォルトレイアウトを使用

#### ログイン画面

```
+----------------------------------+
|                                  |
|        リミメシ管理画面          |
|                                  |
|  [メールアドレス]                |
|  [パスワード]                    |
|  [ログイン]                      |
|                                  |
+----------------------------------+
```

#### チェーン店一覧

```
+----------------------------------+
| リミメシ管理画面          [ログアウト] |
+----------------------------------+
| チェーン店 | メニュー | タグ        |
+----------------------------------+
| [新規作成]                       |
+----------------------------------+
| チェーン名 | 公式サイトURL | [編集] [削除] |
| マクドナルド | https://... | [編集] [削除] |
| スターバックス | https://... | [編集] [削除] |
| ...                              |
+----------------------------------+
```

#### メニュー一覧

```
+----------------------------------+
| リミメシ管理画面          [ログアウト] |
+----------------------------------+
| チェーン店 | メニュー | タグ        |
+----------------------------------+
| [新規作成]                       |
+----------------------------------+
| メニュー名 | チェーン名 | メニュータイプ | ステータス | [編集] [削除] |
| 月見バーガー | マクドナルド | 期間限定 | 販売中 | [編集] [削除] |
| グラコロ | マクドナルド | 期間限定 | 販売終了 | [編集] [削除] |
| ...                              |
+----------------------------------+
```

#### メニュー編集

```
+----------------------------------+
| リミメシ管理画面          [ログアウト] |
+----------------------------------+
| メニュー編集                     |
+----------------------------------+
| 所属チェーン：[マクドナルド▼]    |
| メニュー名：[月見バーガー]       |
| 説明：[...]                      |
| 価格：[390]円                    |
| 画像URL：[...]                   |
| 画像タイプ：[テンプレート▼]      |
| メニュータイプ：[期間限定▼]      |
| 販売開始日：[2025/9/1]           |
| 販売終了日：[2025/10/31]         |
| ステータス：[販売中▼]            |
| [保存] [キャンセル]              |
+----------------------------------+
```

### 5-2. インタラクション

- React Adminのデフォルトインタラクションを使用
- 削除時は確認ダイアログが表示される
- 保存・削除操作はリアルタイムにFirestoreに反映される

### 5-3. デザイン方針

- React Adminのデフォルトテーマを使用（Material-UI）
- シンプルで視認性の高いデザイン
- カラーコントラストWCAG2.1 AA準拠

---

## 6. テスト計画（Test Plan）

### 6-1. 単体テスト

#### 対象

- Custom Claims設定ロジック
- Security Rulesのテスト（Firebase Emulator）

#### ツール

- Jest、Vitest
- Firebase Emulator（Firestore、Auth）

#### テストケース例

```typescript
describe('Security Rules', () => {
  it('管理者はチェーン店を作成できる', async () => {
    // テスト実装
  });

  it('一般ユーザーはチェーン店を作成できない', async () => {
    // テスト実装
  });

  it('未認証ユーザーはチェーン店を作成できない', async () => {
    // テスト実装
  });
});
```

### 6-2. E2Eテスト

#### 対象

- ログイン
- チェーン店の新規登録・編集・削除
- メニューの新規登録・編集・削除
- タグの新規登録・編集・削除

#### ツール

- Playwright、Cypress
- Firebase Emulator（Firestore、Auth）

#### テストシナリオ例

```typescript
test('管理者がログインできる', async ({ page }) => {
  await page.goto('https://admin.limimeshi.com');
  await page.fill('[name=email]', 'admin@example.com');
  await page.fill('[name=password]', 'password');
  await page.click('button:text("ログイン")');
  await expect(page).toHaveURL('https://admin.limimeshi.com/#/chains');
});

test('チェーン店を登録できる', async ({ page }) => {
  await page.goto('https://admin.limimeshi.com/#/chains/create');
  await page.fill('[name=name]', 'テストチェーン');
  await page.click('button:text("保存")');
  await expect(page).toHaveURL('https://admin.limimeshi.com/#/chains');
});
```

### 6-3. 負荷テスト

Phase2では不要（管理者のみ使用）

### 6-4. アクセシビリティテスト

#### 対象

- セマンティックHTML
- キーボード操作
- カラーコントラスト

#### ツール

- axe DevTools
- Lighthouse

#### 確認項目

- [ ] セマンティックHTMLを使用（React Adminのデフォルト）
- [ ] キーボード操作が可能（Tab、Enter、Escapeキー）
- [ ] カラーコントラストがWCAG2.1 AA準拠

### 6-5. テストカバレッジ目標

- 単体テスト：70%以上
- E2Eテスト：主要なCRUD操作（5シナリオ以上）
- アクセシビリティテスト：Lighthouse スコア90以上

---

## 7. 非機能要件（Non-Functional Requirements）

### 7-1. パフォーマンス

- ページ読み込み時間：3秒以内
- 保存・削除操作：1秒以内
- 一覧ページ：ページネーション実装（1ページあたり50件程度）

### 7-2. スケーラビリティ

- 管理者数：Phase2では1名、Phase3以降で複数名想定
- React Adminのデフォルト機能で十分

### 7-3. セキュリティ

- 管理者認証（メール + パスワード、Custom Claims）
- Security Rulesで一般ユーザーのアクセスを制限
- 2要素認証を有効化（Phase2以降）

### 7-4. 信頼性

- 削除確認ダイアログで誤操作防止
- エラーハンドリング：保存エラー、削除エラー
- Firestoreの自動バックアップに依存

### 7-5. 保守性

- TypeScriptで型安全性を確保
- React Adminのデフォルト機能を最大限活用
- カスタマイズは最小限（保守性重視）

---

## 8. 残課題（TBD）

### Phase1で検討

- [ ] 入力補助機能の詳細設計（過去の類似メニューからの入力候補表示）
- [ ] Security Rulesの詳細化（data-model/security-rules.md）
- [ ] Custom Claims設定方法の詳細化

### Phase2で実装

- [ ] テンプレアイコンの管理方法（Phase2でテンプレアイコン作成）
- [ ] 管理者アカウントの作成方法（Cloud Functions or 手動）
- [ ] 2要素認証の有効化（Phase2リリース後）
- [ ] 負荷テスト不要確認（管理者のみ使用）

### Phase3以降で追加

- [ ] ユーザー管理（一般ユーザーの一覧表示、削除、統計情報）
- [ ] レビュー管理（一覧表示、削除、モデレーション）
- [ ] 販売終了メニューの一括更新機能
- [ ] メニュー登録の効率化（公式サイトのURLを入力すると情報を抽出する機能）
- [ ] ダッシュボード（KPI、アクセス解析、コスト監視）

---

## 9. 参考資料

- [planning/first-idea.md](../planning/first-idea.md)：サービス全体像、ペルソナ、データモデル
- [planning/inception-deck.md](../planning/inception-deck.md)：優先順位、トレードオフ
- [React Admin公式ドキュメント](https://marmelab.com/react-admin/)
- [react-admin-firebase](https://github.com/benwinding/react-admin-firebase)
- [WRITING_STYLE_GUIDE.md](../WRITING_STYLE_GUIDE.md)：ドキュメント記述ルール

---

## 更新履歴

- 2025/10/26：初版作成
