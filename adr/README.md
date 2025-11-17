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

## 作成済み

- [ADR-001: Use Firebase for backend](./001-use-firebase-for-backend.md)
- [ADR-002: Adopt multi-repository structure](./002-adopt-multi-repository-structure.md)
- [ADR-003: Use React Admin for admin panel](./003-use-react-admin-for-admin-panel.md)
- [ADR-004: Use manual data entry for Phase 2](./004-use-manual-data-entry-for-phase2.md)
- [ADR-005: Deploy using Firebase Hosting multi-site](./005-deploy-using-firebase-hosting-multi-site.md)

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
