# GitHub Spec Kit テンプレート

このディレクトリは新規リポジトリに `.specify/` ディレクトリを導入するためのテンプレート

## GitHub Spec Kitとは

**GitHub Spec Kit** は仕様駆動開発（Spec-Driven Development）を実践するためのフレームワーク

**公式リポジトリ**: [github/spec-kit](https://github.com/github/spec-kit)

### 哲学

- **仕様が王様、コードは従者**: 仕様書からコードを生成する
- **仕様と実装のギャップをゼロに**: 仕様変更は必ずコードに反映
- **テスト駆動開発（TDD）**: テストを先に書いてから実装

### ワークフロー

```
/speckit-specify → /speckit-clarify → /speckit-plan → /speckit-tasks → /speckit-implement
```

1. **specify**: 自然言語の機能説明から仕様書（spec.md）を生成
2. **clarify**: 仕様書の曖昧な箇所を質問で解決
3. **plan**: 技術設計・アーキテクチャを策定（plan.md）
4. **tasks**: 実装タスクリスト（tasks.md）を生成
5. **implement**: タスクを順番に実行

## ディレクトリ構造

```
.specify/
├── memory/
│   └── constitution.md  # プロジェクトの憲法（開発原則）
├── specs/               # 機能仕様書（機能ごとにサブディレクトリ）
│   └── .gitkeep
├── templates/           # 仕様書・計画書のテンプレート
│   ├── spec-template.md
│   ├── plan-template.md
│   ├── tasks-template.md
│   ├── checklist-template.md
│   └── agent-file-template.md
└── .claude/
    └── commands/        # Custom Slash Commands（speckit-*.md）
```

## 新規リポジトリへの導入

1. このディレクトリ内容を新規リポジトリの `.specify/` にコピー
2. `memory/constitution.md` をプロジェクトに合わせてカスタマイズ
3. Claude Codeの `/speckit-specify` コマンドで機能仕様書を作成開始

## constitution.mdについて

`memory/constitution.md` はプロジェクトの**憲法**

- 開発原則（Test-First、Simplicity など）
- 技術選定方針
- 品質基準
- ガバナンスルール

を定義。全ての実装・設計判断はこの憲法に準拠する。

**マスターファイル**: [docs/governance/constitution.md](../governance/constitution.md)

新規リポジトリでは初期値としてマスターをコピーし、その後は各リポジトリで独立管理

## 関連ドキュメント

- [limimeshi-docsガバナンス](../governance/)
- [setup-new-repo.mdコマンド](../../.claude/commands/setup-new-repo.md)
- [GitHub Spec Kit公式](https://github.com/github/spec-kit)

---

**Version**: 1.0.0 | **Created**: 2025/12/05
