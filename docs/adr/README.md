# Architecture Decision Records（ADR）

技術選定の背景と理由を記録するドキュメントを格納しています。

## ADRとは

Architecture Decision Records（ADR）は、アーキテクチャに関する重要な決定とその理由を記録するドキュメント

**出典**: Michael Nygard "[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)" (2011)

### ADRの構成（Michael Nygard形式）

1. **Status**: proposed, accepted, deprecated, superseded等
2. **Context**: なぜこの決定が必要か
3. **Decision**: 何を選択したか
4. **Consequences**: この決定がもたらす影響（良い面も悪い面も）

## 作成済み（共通ADR）

- [ADR-001: Use Firebase for backend](./001-use-firebase-for-backend.md)
- [ADR-002: Adopt multi-repository structure](./002-adopt-multi-repository-structure.md)
- [ADR-004: Use manual data entry for Phase 2](./004-use-manual-data-entry-for-phase2.md)
- [ADR-005: Deploy using Firebase Hosting multi-site](./005-deploy-using-firebase-hosting-multi-site.md)

## 固有ADR（各リポジトリに配置）

- limimeshi-admin: [ADR-001: Use React Admin for admin panel](https://github.com/shg25/limimeshi-admin/blob/main/.specify/adr/001-use-react-admin-for-admin-panel.md)

## 命名規則

### ファイル名

`NNN-short-title.md`形式で命名
- **NNN**: 連番（001, 002, 003...）
- **short-title**: 短いタイトル（kebab-case）

**例**: `001-use-firebase-for-backend.md`

### タイトル命名規則

**形式**: Present tense imperative verb phrase（現在形の命令形動詞句）

**典型的なパターン**:
- `Use [technology] for [purpose]`
- `Adopt [approach]`
- `Choose [option]`
- `Deploy using [method]`

**具体例**:
- `Use Firebase for backend`
- `Adopt multi-repository structure`
- `Use React Admin for admin panel`
- `Deploy using Firebase Hosting multi-site`

**理由**:
- 決定内容が明確
- 読みやすく、スキャンしやすい
- コミットメッセージと形式が一致

**出典**: [architecture-decision-record](https://github.com/joelparkerhenderson/architecture-decision-record) (Joel Parker Henderson)

---

## ADR配置方針

ADRは影響範囲に応じて配置場所を分ける

| 配置場所 | 対象 | 例 |
|----------|------|-----|
| limimeshi-docs/adr/ | 複数リポジトリに影響する決定 | Firebase選定、マルチリポジトリ構成 |
| limimeshi-admin/.specify/adr/ | 管理画面固有の決定 | React Admin選定 |
| limimeshi-android/adr/ | Android固有の決定 | Jetpack Compose選定、アーキテクチャパターン |

### 判断基準

- **共通ADR（このリポジトリ）**: 複数のリポジトリで参照される技術決定
- **固有ADR（各リポジトリ）**: そのリポジトリの実装にのみ関係する技術決定

### 番号の連番

- このリポジトリ（limimeshi-docs）の連番と、各リポジトリの連番は独立
- 例: limimeshi-docs/adr/005-xxx.md と limimeshi-android/adr/001-xxx.md は共存可能
