# 新規リポジトリセットアップテンプレート

このディレクトリの内容をまるごとコピーすることで、新規limimeshiリポジトリの初期セットアップが完了

## 使い方

```bash
# 新規リポジトリのルートで実行
cp -r <limimeshi-docs>/template/setup-new-repo/* .
cp -r <limimeshi-docs>/template/setup-new-repo/.* . 2>/dev/null || true
```

## ディレクトリ構成

```
setup-new-repo/
├── .specify/              # GitHub Spec Kit（仕様駆動開発）
│   ├── memory/
│   │   └── constitution.md  # プロジェクトの憲法（カスタマイズ可）
│   ├── specs/               # 機能仕様書（空）
│   ├── templates/           # 仕様書テンプレート
│   └── .claude/
│       └── commands/        # speckit-*.md コマンド
├── .claude/               # Claude Code設定
│   ├── commands/
│   │   └── suggest-claude-md.md
│   ├── skills/
│   │   ├── security-check.md
│   │   └── style-guide-check.md
│   └── settings.json
└── docs/                  # ガバナンスドキュメント
    ├── governance/
    │   ├── docs-style-guide.md
    │   └── shared-rules.md
    ├── adr/
    │   └── README.md
    ├── roadmap.md         # リポジトリ固有のロードマップ
    └── CHANGELOG.md       # 変更履歴
```

## コピー後の作業

1. **constitution.md**: プロジェクトに合わせてカスタマイズ
2. **roadmap.md**: `[REPOSITORY_NAME]`を実際のリポジトリ名に変更
3. **CLAUDE.md**: プロジェクト固有の情報を追記（任意）

## 継続同期

以下のファイルはlimimeshi-docsから継続同期される：
- `.claude/` 配下（speckit-*以外）
- `docs/governance/` 配下

同期は `/sync-shared-rules` コマンドで実行

## 関連

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [limimeshi-docs](https://github.com/shg25/limimeshi-docs)
