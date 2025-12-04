# 共有マスターファイル

このディレクトリには、複数のlimimeshiリポジトリで共有されるマスターファイルを配置

## ディレクトリ構造

```
shared/
└── setup-new-repo/           # 各リポジトリからシンボリックリンクで参照
    ├── .claude/              # Claude Code設定
    │   ├── commands/
    │   │   └── suggest-claude-md.md
    │   ├── skills/
    │   │   ├── security-check.md
    │   │   └── style-guide-check.md
    │   └── settings.json
    ├── .specify/             # Spec Kitファイル
    │   ├── .claude/commands/
    │   │   └── speckit-*.md  (8ファイル)
    │   └── templates/
    │       └── *-template.md (5ファイル)
    └── docs/
        └── governance/       # ガバナンスドキュメント
            ├── docs-style-guide.md
            └── shared-rules.md
```

## 使用方法

### 他リポジトリからの参照

各リポジトリ（limimeshi-admin, limimeshi-android等）は、このディレクトリへの**シンボリックリンク**を持つ：

```
limimeshi-admin/.claude/settings.json
    → ../limimeshi-docs/shared/setup-new-repo/.claude/settings.json
```

### 自動同期

シンボリックリンク方式のため、**このディレクトリを編集すれば全リポジトリに自動反映**。
手動での同期作業は不要。

### 新規リポジトリ作成

`/setup-new-repo` コマンドでシンボリックリンクを作成

### ファイル追加時

shared/にファイルを追加した場合のみ、各リポジトリで `/sync-shared-rules` を実行してシンボリックリンクを追加

## 前提条件

全リポジトリは同じ親ディレクトリに配置する必要がある：
```
parent-directory/
├── limimeshi-docs/       ← このリポジトリ（マスター）
├── limimeshi-admin/
└── limimeshi-android/
```

## 注意事項

- このディレクトリ内のファイルがマスター
- 変更は必ずこのディレクトリ内で行う
- 他リポジトリのシンボリックリンクは編集しない（編集してもこちらに反映される）
