# Implementation Plan: 管理画面（Admin Panel）

**Branch**: `001-admin-panel` | **Date**: 2025-11-16 | **Spec**: [spec.md](./spec.md)    
**Input**: Feature specification from `/specs/001-admin-panel/spec.md`    

## Summary

管理者がチェーン店と期間限定メニューを手動で登録・編集・削除できる管理画面を提供する。React Adminをベースとし、Firebase Authentication（Custom Claims）による管理者認証を実装。チェーン店16店舗、各店舗あたり平均10件のメニューを管理する想定。Phase2 MVPの最優先機能として、手動データ登録の基盤を提供する。

## Technical Context

**Language/Version**: TypeScript 5.x, React 18.x  
**Primary Dependencies**: React Admin 4.x, Firebase SDK 10.x, Vite 5.x  
**Storage**: Firestore (NoSQL document database)  
**Testing**: Vitest, React Testing Library, Playwright (E2E)  
**Target Platform**: Web (Desktop primary, モバイル最適化は優先度低)  
**Project Type**: Web application (frontend + backend)  
**Performance Goals**: ページ読み込み時間 3秒以内、保存・削除操作 1秒以内  
**Constraints**: モバイル4G環境での動作、Phase2は1名運用（同時編集考慮不要）  
**Scale/Scope**: 16チェーン店、平均160メニュー、1名の管理者（Phase2）  

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Spec-Driven ✅ PASS
- spec.md完成済み（2025/11/14）
- plan.md作成中（このファイル）
- tasks.md は `/speckit-tasks` で生成予定

### II. Test-First ✅ PASS
- 各User Storyに対応するAcceptance Scenariosが明確
- テスト実装は tasks.md で定義予定

### III. Simplicity ✅ PASS
- React Adminのデフォルト機能を活用（不要なラッパー作成なし）
- Firestoreの標準クエリを使用（複雑な抽象化なし）
- Phase2では最小機能に限定（タグ管理・入力補助機能は Out of Scope）

### IV. Firebase-First ✅ PASS
- Firebase Authentication: 管理者認証（Custom Claims）
- Firestore: チェーン店・メニューデータ
- Firebase Hosting: デプロイ（将来）

### V. Legal Risk Zero ✅ PASS
- 完全手動データ登録（スクレイピングなし）
- チェーン店の公式URL・ロゴURLは手動入力のみ

### VI. Mobile & Performance First ⚠️ PARTIAL
- パフォーマンス目標: ページ読み込み3秒以内、保存1秒以内
- モバイル最適化は優先度低（管理画面はPC主体）
- **判定**: Phase2ではPC優先、Phase3でモバイル最適化検討

**GATE RESULT**: ✅ PASS（モバイル最適化はPhase3に延期、Phase2はPC優先で承認）

## Project Structure

### Documentation (this feature)

```text
specs/001-admin-panel/
├── spec.md              # Feature specification (completed)
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (below)
├── data-model.md        # Phase 1 output (below)
├── quickstart.md        # Phase 1 output (below)
├── contracts/           # Phase 1 output (below)
│   ├── firestore-schema.md
│   └── react-admin-data-provider.md
├── checklists/
│   └── requirements.md  # Spec quality checklist (completed)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
limimeshi-admin/         # 管理画面リポジトリ（別リポジトリ）
├── src/
│   ├── components/      # React Admin カスタムコンポーネント
│   │   ├── chains/      # チェーン店リソース
│   │   │   ├── ChainList.tsx
│   │   │   ├── ChainEdit.tsx
│   │   │   └── ChainCreate.tsx
│   │   ├── menus/       # メニューリソース
│   │   │   ├── MenuList.tsx
│   │   │   ├── MenuEdit.tsx
│   │   │   ├── MenuCreate.tsx
│   │   │   └── StatusField.tsx  # ステータス自動判定
│   │   └── auth/        # 認証コンポーネント
│   │       └── LoginPage.tsx
│   ├── providers/       # React Admin データプロバイダー
│   │   ├── dataProvider.ts      # Firestore データプロバイダー
│   │   └── authProvider.ts      # Firebase Authentication プロバイダー
│   ├── lib/             # ユーティリティ
│   │   ├── firebase.ts  # Firebase初期化
│   │   └── statusUtils.ts  # ステータス自動判定ロジック
│   ├── App.tsx          # React Adminアプリケーション
│   └── main.tsx         # エントリーポイント
├── tests/
│   ├── unit/            # ユニットテスト（Vitest）
│   │   └── statusUtils.test.ts
│   ├── integration/     # 統合テスト（React Testing Library）
│   │   ├── ChainList.test.tsx
│   │   └── MenuList.test.tsx
│   └── e2e/             # E2Eテスト（Playwright）
│       ├── auth.spec.ts
│       ├── chains.spec.ts
│       └── menus.spec.ts
├── firestore.rules      # Firestoreセキュリティルール
├── firestore.indexes.json  # Firestoreインデックス
├── vite.config.ts       # Vite設定
├── package.json
└── README.md

limimeshi-docs/          # このリポジトリ（ドキュメント専用）
└── specs/001-admin-panel/  # 上記参照
```

**Structure Decision**:
- 管理画面は別リポジトリ `limimeshi-admin` として実装（一般向けWebアプリ `limimeshi-web` と分離）
- Web application構造を採用（frontend: React Admin、backend: Firestore + Cloud Functions）
- React Adminの標準ディレクトリ構造に従う

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

該当なし（全てのConstitution Checkに合格）

---

## Phase 0: Research & Technology Decisions

### Research Tasks

1. **React Admin 4.x Best Practices**
   - 目的: React Adminの標準的な使い方を理解
   - 調査内容: データプロバイダー、認証プロバイダー、カスタムフィールドの実装パターン
   - 成果物: research.md に記録

2. **Firestore Data Provider for React Admin**
   - 目的: FirestoreとReact Adminの統合パターン
   - 調査内容: 既存ライブラリ（ra-data-firestore-client等）vs カスタム実装
   - 成果物: research.md に決定事項を記録

3. **Firebase Authentication with Custom Claims**
   - 目的: 管理者認証の実装方法
   - 調査内容: Custom Claimsの設定方法、Cloud Functionsでの権限設定
   - 成果物: research.md に実装手順を記録

4. **Firestore Security Rules for Admin Panel**
   - 目的: 管理者のみがアクセスできるセキュリティルール
   - 調査内容: Custom Claimsを使った権限チェック
   - 成果物: research.md にルール例を記録

5. **Vite + React + TypeScript Setup**
   - 目的: 開発環境のセットアップ
   - 調査内容: Viteの設定、TypeScript設定、ESLint/Prettier設定
   - 成果物: research.md に設定例を記録

### Research Agents

各研究タスクについて、以下の形式で調査結果を `research.md` に記録：

```markdown
## [Technology/Pattern Name]

### Decision
[採用した技術・パターン]

### Rationale
[選択理由]

### Alternatives Considered
[検討した代替案]

### Implementation Notes
[実装時の注意点]
```

---

## Phase 1: Design & Contracts

### 1.1 Data Model Design

**Output**: `data-model.md`

#### Entities

**Chain（チェーン店）**
- チェーンID（自動生成）
- チェーン名（必須）
- ふりがな（必須、50音順ソート用）
- 公式サイトURL（任意）
- ロゴ画像URL（任意）
- お気に入り登録数（集約データ、初期値0）
- 作成日時（自動）
- 更新日時（自動）

**Menu（メニュー）**
- メニューID（自動生成）
- 所属チェーンID（外部キー、必須）
- メニュー名（必須）
- 説明（任意）
- 価格（任意）
- X Post URL（任意）
- 販売開始日時（必須）
- 販売終了日時（任意）
- ステータス（計算フィールド、保存しない）
- 作成日時（自動）
- 更新日時（自動）

**Admin（管理者）**
- ユーザーID（Firebase Authentication UID）
- 表示名（任意）
- メールアドレス（必須）
- Custom Claims: `{ admin: true }`
- 作成日時（自動）

#### Relationships

```
Chain (1) ─── (N) Menu
  ↑
  └─ ふりがなでソート、お気に入り登録数を集約

Admin ─── (認証) ─── System
  ↑
  └─ Custom Claims で権限管理
```

#### Validation Rules

- チェーン名: 必須、最大100文字
- ふりがな: 必須、最大100文字、ひらがなのみ
- メニュー名: 必須、最大100文字
- 販売開始日時: 必須、ISO 8601形式
- 販売終了日時: 任意、販売開始日時より後であること
- 価格: 任意、0以上の整数

#### State Transitions

**メニューのステータス自動判定**:
- 予定: `current_time < sale_start_time`
- 発売から◯日経過: `sale_end_time == null && (current_time - sale_start_time) < 1 month`
- 発売から◯ヶ月以上経過: `sale_end_time == null && (current_time - sale_start_time) >= 1 month`
- 販売終了: `current_time > sale_end_time`

### 1.2 API Contracts

**Output**: `contracts/`

#### Firestore Collections

**contracts/firestore-schema.md**:
```
/chains/{chainId}
  - name: string
  - furigana: string
  - officialUrl?: string
  - logoUrl?: string
  - favoriteCount: number (default: 0)
  - createdAt: timestamp
  - updatedAt: timestamp

/menus/{menuId}
  - chainId: reference to /chains/{chainId}
  - name: string
  - description?: string
  - price?: number
  - xPostUrl?: string
  - saleStartTime: timestamp
  - saleEndTime?: timestamp
  - createdAt: timestamp
  - updatedAt: timestamp

/admins/{adminId}
  - displayName?: string
  - email: string
  - createdAt: timestamp
```

#### React Admin Data Provider Interface

**contracts/react-admin-data-provider.md**:
```typescript
interface DataProvider {
  // チェーン店
  getList(resource: 'chains', params: GetListParams): Promise<GetListResult>
  getOne(resource: 'chains', params: GetOneParams): Promise<GetOneResult>
  create(resource: 'chains', params: CreateParams): Promise<CreateResult>
  update(resource: 'chains', params: UpdateParams): Promise<UpdateResult>
  // Phase2では削除機能なし（deleteは実装しない）

  // メニュー
  getList(resource: 'menus', params: GetListParams): Promise<GetListResult>
  getOne(resource: 'menus', params: GetOneParams): Promise<GetOneResult>
  create(resource: 'menus', params: CreateParams): Promise<CreateResult>
  update(resource: 'menus', params: UpdateParams): Promise<UpdateResult>
  delete(resource: 'menus', params: DeleteParams): Promise<DeleteResult>
}

// ステータスは計算フィールドとして getList/getOne で追加
interface MenuWithStatus extends Menu {
  status: 'scheduled' | 'days_elapsed' | 'months_elapsed' | 'ended'
  statusDisplay: string  // 表示用文字列
}
```

### 1.3 Agent Context Update

**Script**: `bash scripts/bash/update-agent-context.sh`

エージェントコンテキストファイル（CLAUDE.md）に以下の技術情報を追加：
- React Admin 4.x
- Firestore data provider pattern
- Firebase Authentication with Custom Claims
- Vite + React + TypeScript

### 1.4 Quickstart Guide

**Output**: `quickstart.md`

開発環境のセットアップ手順：
1. Node.js 20.x インストール
2. `git clone` して `npm install`
3. Firebase プロジェクト作成
4. `.env.local` に Firebase 設定
5. `npm run dev` で開発サーバー起動
6. 管理者ユーザーを Firebase Console で作成（Custom Claims設定）

---

## Phase 2: Task Generation

**Command**: `/speckit-tasks`

このフェーズは `/speckit-plan` の範囲外。`/speckit-tasks` コマンドで `tasks.md` を生成する。

---

## Notes

- **Phase2 MVP範囲**: チェーン店管理、メニュー管理、管理者認証
- **Out of Scope**: タグ管理、入力補助機能、チェーン店削除機能
- **依存関係**: なし（最初に実装する機能）
- **実装リポジトリ**: `limimeshi-admin`（別リポジトリ、このドキュメントリポジトリとは分離）
