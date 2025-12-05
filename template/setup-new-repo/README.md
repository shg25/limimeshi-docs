# 新規リポジトリセットアップテンプレート

このディレクトリはコピー対象の実体ファイルのみを含む

## このディレクトリの内容

コピー対象（各リポジトリ固有）：
- `.specify/README.md` - Spec Kit使い方ガイド
- `.specify/memory/constitution.md` - プロジェクトの憲法
- `.specify/specs/.gitkeep` - 機能仕様書ディレクトリ
- `docs/README.md` - ドキュメントディレクトリ説明
- `docs/adr/README.md` - ADRディレクトリ
- `docs/governance/README.md` - ガバナンスルール説明
- `docs/roadmap.md` - ロードマップ
- `docs/CHANGELOG.md` - 変更履歴

※ シンボリックリンク対象ファイルは `shared/setup-new-repo/` に配置

## セットアップ方法

limimeshi-docsリポジトリから `/setup-new-repo [リポジトリ名]` を実行

```bash
# 1. limimeshi-docsをclone
git clone https://github.com/shg25/limimeshi-docs.git

# 2. 同じディレクトリで新規リポジトリを作成
mkdir new-repo && cd new-repo
git init

# 3. limimeshi-docsに移動してセットアップ実行
cd ../limimeshi-docs
# /setup-new-repo new-repo を実行
```

## ディレクトリ構成

```
new-repo/
├── .specify/              # GitHub Spec Kit（仕様駆動開発）
│   ├── memory/
│   │   └── constitution.md  # 実体（カスタマイズ可）
│   ├── specs/               # 実体（リポジトリ固有）
│   ├── templates/           # → limimeshi-docs/shared/...
│   └── .claude/
│       └── commands/        # → limimeshi-docs/shared/...
├── .claude/               # Claude Code設定
│   ├── commands/          # → limimeshi-docs/shared/...
│   ├── skills/            # → limimeshi-docs/shared/...
│   └── settings.json      # → limimeshi-docs/shared/...
└── docs/                  # ガバナンスドキュメント
    ├── governance/        # → limimeshi-docs/shared/...
    ├── adr/
    │   └── README.md      # 実体（リポジトリ固有）
    ├── roadmap.md         # 実体（リポジトリ固有）
    └── CHANGELOG.md       # 実体（リポジトリ固有）
```

※ `→` 付きはlimimeshi-docs/shared/へのシンボリックリンク

## 前提条件

全limimeshiリポジトリは同じ親ディレクトリに配置：
```
parent-directory/
├── limimeshi-docs/    ← 必須（マスター）
├── limimeshi-admin/
├── limimeshi-android/
└── new-repo/
```

## 継続同期

シンボリックリンクのファイルは自動同期。
limimeshi-docs/shared/を編集すれば全リポジトリに反映される。

**同期対象外（各リポジトリ固有）**:
- constitution.md
- specs/
- roadmap.md
- CHANGELOG.md
- adr/

## 関連

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [limimeshi-docs](https://github.com/shg25/limimeshi-docs)
