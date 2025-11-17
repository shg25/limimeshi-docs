# Architecture Decision Records（ADR）

技術選定の背景と理由を記録するドキュメントを格納しています。

## ADRとは

Architecture Decision Records（ADR）は、アーキテクチャに関する重要な決定とその理由を記録するドキュメントです。

### ADRの構成
1. ステータス（Status）：提案中、承認済み、非推奨など
2. コンテキスト（Context）：なぜこの決定が必要か
3. 決定（Decision）：何を選択したか
4. 結果（Consequences）：この決定がもたらす影響

## 作成予定

Phase1で以下のADRを作成します：
- [ ] `001-why-firebase.md`：なぜFirebaseを選んだか
- [ ] `002-monorepo-vs-multirepo.md`：モノレポ vs マルチリポ
- [ ] `003-react-admin.md`：React Adminを選んだ理由
- [ ] `004-manual-data-update.md`：データ更新の運用方針（完全手動運用の選定理由）
- [x] `005-firebase-hosting-multi-site.md`：Firebase Hosting構成とインフラ管理戦略

## 命名規則

`NNN-short-title.md`形式で命名します。
- NNN：連番（001, 002, 003...）
- short-title：短いタイトル（kebab-case）
