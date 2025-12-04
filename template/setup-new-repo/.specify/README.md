# GitHub Spec Kit（仕様駆動開発）

仕様書からコードを生成する開発手法を支援するディレクトリ

## Spec Kitとは

**Spec-Driven Development**（仕様駆動開発）を実践するためのフレームワーク

- 仕様が王様、コードは従者
- 仕様書を先に書き、それに基づいてコードを生成
- 仕様と実装のギャップをゼロにする

**参考**: [GitHub Spec Kit](https://github.com/github/spec-kit)

## ディレクトリ構成

```
.specify/
├── memory/
│   └── constitution.md    # プロジェクトの憲法（実体、カスタマイズ可）
├── specs/                 # 機能仕様書（実体、リポジトリ固有）
│   └── NNN-feature-name/
│       ├── spec.md        # 仕様書本体
│       ├── plan.md        # 実装計画
│       ├── tasks.md       # タスク一覧
│       └── checklist.md   # 完了チェックリスト
├── templates/             # テンプレート（→ limimeshi-docs/shared/）
└── .claude/commands/      # speckit-*コマンド（→ limimeshi-docs/shared/）
```

## 基本ワークフロー

### 1. 仕様作成

```
/speckit-specify NNN-feature-name
```

`specs/NNN-feature-name/spec.md`を作成

### 2. 仕様分析・質問

```
/speckit-analyze NNN-feature-name
/speckit-clarify NNN-feature-name
```

仕様の曖昧な点を洗い出し、質問を整理

### 3. 実装計画

```
/speckit-plan NNN-feature-name
```

`plan.md`と`tasks.md`を作成

### 4. 実装

```
/speckit-implement NNN-feature-name
```

タスクに従って実装を進める

### 5. 完了確認

```
/speckit-checklist NNN-feature-name
```

`checklist.md`で完了状態を確認

## コマンド一覧

| コマンド | 説明 |
|---------|------|
| `/speckit-specify` | 仕様書を作成 |
| `/speckit-analyze` | 仕様を分析 |
| `/speckit-clarify` | 不明点を質問形式で整理 |
| `/speckit-plan` | 実装計画を作成 |
| `/speckit-tasks` | タスク一覧を更新 |
| `/speckit-implement` | 実装を進める |
| `/speckit-checklist` | 完了チェックリストを作成・確認 |
| `/speckit-constitution` | constitution.mdを参照・更新 |

## 仕様書の命名規則

`NNN-feature-name`形式で命名
- **NNN**: 連番（001, 002, 003...）
- **feature-name**: 機能名（kebab-case）

例: `001-admin-panel`, `002-chain-list`, `003-favorites`

## constitution.md

プロジェクトの憲法。全ての実装・設計はこれに準拠する。

主な内容：
- 開発原則（Test-First、Firebase-First等）
- 技術選定方針
- 品質基準
- 禁止事項

## 関連

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [limimeshi-docs](https://github.com/shg25/limimeshi-docs)
