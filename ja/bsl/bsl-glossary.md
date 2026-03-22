# BSL Glossary

**Version:** v0.5.1

## Scope / SSOT Declaration

本ファイルは BSL（仕様・運用ガイド）に固有の用語集であり、BSL 仕様語彙の正本（SSOT）とする。  
Core 概念の定義は between/（Core Glossary および Core 本文）を参照し、ここでは再定義しない。

- Scope: BSL（既定）
- SSOT: docs/（BSL 本文・BSL Annex）
- Core 参照: 用語ごとに `Depends on (Core)` を必須とする（該当する場合）

## Entry Format（記載形式）

各エントリは次の最小要素を持つ。

- Term: 用語名
- Scope: BSL
- Definition: 1〜2文（仕様としての意味）
- Spec ref (BSL): 本用語の正本位置（章/節/Annex）
- Depends on (Core): 参照する Core 用語（該当する場合）
- Notes: 運用上の注意（任意）
- Status: active / deprecated / reserved（任意）
- Aliases: 別名（任意）

---

## Terms

### Evidence（BSL における仕様化）

- Scope: BSL
- Definition: Core の Evidence（Trace）を、BSL で機械可読な構造として表現するための仕様。受動的観測と能動的記録の両方を含む。
- Spec ref (BSL): Chapter 4. Evidence
- Depends on (Core): Evidence (Trace), AX-E1, AX-E2
- Notes: BSL では Evidence の分類（trace_type）を Condition 内で表現する。Evidence 自体は因果・意図を表さない（Core AX-E1）。
- Status: active

### Reading

- Scope: BSL
- Definition: Core の Evidence.Element（Reading）を BSL 表現したもの。その瞬間に得た／与えた値を表す。
- Spec ref (BSL): Chapter 4. Evidence / 4.1 Reading
- Depends on (Core): Evidence.Element (Reading)
- Notes: 受動的観測（センサ値、測定結果）と能動的記録（発話、制御出力、API応答）の両方を含む。能動的記録の結果として生じた状態変化は Reading に含めず、Behavior/Flow で表現する（Core LEM-E3）。ActionTrace/EffectDelta に対するドメイン別の便宜名（Alias）を定義してよい。
- Status: active

### ToolCall（Alias）

- Definition: ActionTrace（行為の記録）のうち、外部ツール呼び出しを表す便宜名。
- Representation: Reading.value.kind="action_trace" かつ Reading.value.action_type="tool_call"
- Non-implication: 成功・反映・権限OKを含意しない（AX-E1/LEM-E3）。
- Status: active

### ToolEffect（Alias）

- Definition: EffectDelta（観測された状態変化）のうち、外部ツール由来として扱う便宜名。
- Representation: Behavior/Flow 側の差分（Evidenceではない）。
- Non-implication: 正当性・採否は含意しない（評価フレームΦに依存）。
- Status: active

### References（related policies）

- docs/policies/evidence-class-declaration.md
- docs/policies/external-actions-deny-by-default.md

### Condition

- Scope: BSL
- Definition: Core の Evidence.Structure（Condition）を BSL 表現したもの。Reading を意味づける前提条件を表す。
- Spec ref (BSL): Chapter 4. Evidence / 4.2 Condition
- Depends on (Core): Evidence.Structure (Condition)
- Notes: 基準面・治具・姿勢・環境に加え、セッション状態・認証状態なども含む。trace_type フィールドで受動/能動の区別を表現可能。Condition は Reading の解釈に必要な前提に限り、判断結果（成立/不成立、正否、意図推定）を含めない。
- Status: active

### Ordering

- Scope: BSL
- Definition: Core の Evidence.Basis（Ordering）を BSL 表現したもの。痕跡が生成された順序を表す。
- Spec ref (BSL): Chapter 4. Evidence / 4.3 Ordering
- Depends on (Core): Evidence.Basis (Ordering), DEF-E4
- Notes: Behavior.Sequence（規範的手順）とは異なる。Ordering は記録の並びであり、規範や設計意図を表さない（Core DEF-E4）。
- Status: active

### Evidence Chain

- Scope: BSL
- Definition: Reading / Condition / Ordering を非破壊で蓄積する append-only の証跡構造。
- Spec ref (BSL): Chapter 4. Evidence / 3. Evidence Chain
- Depends on (Core): Sidecar, AX-6
- Notes: Flow / Behavior の Structure を変更せず、外側に蓄積される。
- Status: active

### trace_type

- Scope: BSL
- Definition: Evidence が受動的観測か能動的記録かを区別するための Condition 内フィールド。
- Spec ref (BSL): Chapter 4. Evidence / 4.2 Condition
- Depends on (Core): Evidence (Trace), AX-E2
- Notes: 値は "observation"（受動的観測）または "action"（能動的記録）。区別が不要な場合は省略可。省略時は "observation" を既定としない。省略は「分類しない」を意味する。
- Status: active

### VariationPolicy

- Scope: BSL
- Definition: Variation（許容される揺らぎ）を機械可読に宣言する構造。
- Spec ref (BSL): Annex B. Variation Policy Schema
- Depends on (Core): Variation, Variation Policy, Sidecar
- Notes: Sidecar の append-only 原則に従い、修正は新規 Policy を追加し、旧版を superseded とする。
- Status: active

### PolicyRef

- Scope: BSL
- Definition: 上位構造（例：Reading）から VariationPolicy を参照するための参照形式。
- Spec ref (BSL): Annex B. Variation Policy Schema / 2.5 tolerance の埋め込み形
- Depends on (Core): Variation Policy, Sidecar
- Notes: BSL 本文では参照形式を使用し、インライン定義は Sandboxes/アプリ層の拡張として扱う。
- Status: active

### severity

- Scope: BSL
- Definition: VariationPolicy 逸脱時の重大度を表すフィールド（info / warning / fail）。
- Spec ref (BSL): Annex B. Variation Policy Schema / Data Model
- Depends on (Core): Variation Policy
- Status: active

### status

- Scope: BSL
- Definition: Policy の状態（例：active, superseded）を表すメタ情報。
- Spec ref (BSL): Annex B. Variation Policy Schema / Versioning Policy
- Depends on (Core): Sidecar（append-only）
- Notes: status を導入する場合、superseded_by と組で運用する。
- Status: active

### superseded_by

- Scope: BSL
- Definition: superseded となった Policy が参照すべき新版 Policy ID。
- Spec ref (BSL): Annex B. Variation Policy Schema / Versioning Policy
- Depends on (Core): Sidecar（append-only）
- Status: active

### Space

- Scope: BSL
- Definition: 同一性（Identity）を評価するための前提が閉じている単位。BSL では space_id で識別し、比較は原則として同一 space_id かつ同一 basis_id の内側でのみ許可する。
- Spec ref (BSL): Chapter 1. Core Concepts / 8.1 Space、Chapter 9. Architecture / 6-7
- Depends on (Core): Core 9.2.1（Space の意味論的定義）
- Notes: Space は組織やシステム構成と一致してもしなくてもよい。同一 SSOT でも view/sidecar が異なれば別 Space として扱ってよい（MAY）。
- Status: active

### Mapping

- Scope: BSL
- Definition: Space 間または役割間の接続。接続の実体（API、ETL、イベント、手作業等）に依存しないが、接続が存在するなら mapping_id と mapping_kind で宣言できること。
- Spec ref (BSL): Chapter 1. Core Concepts / 8.2 Mapping、Chapter 9. Architecture / 6.3
- Depends on (Core): Core 9.2.1（Space 間接続の原則）
- Notes: Mapping は「直接参照」や「暗黙の逆流」を正当化しない。
- Status: active

### space_id

- Scope: BSL
- Definition: Space 識別子。比較の許可単位。
- Spec ref (BSL): Chapter 9. Architecture / 6.1
- Depends on (Core): Core 9.2.1
- Status: active

### basis_id

- Scope: BSL
- Definition: 解釈前提の識別子。view および sidecar に必須。basis_id が異なる成果物は直接比較してはならない。
- Spec ref (BSL): Chapter 9. Architecture / 6.2
- Depends on (Core): Meaning Identity, Basis
- Status: active

### trace_id

- Scope: BSL
- Definition: 証跡連鎖の識別子。evidence に必須。ssot/view の意味的変更は trace_id により説明可能でなければならない。
- Spec ref (BSL): Chapter 9. Architecture / 6.2、Chapter 4. Evidence / 7
- Depends on (Core): Evidence, Sidecar
- Status: active

### artifact_role

- Scope: BSL
- Definition: アーティファクトの役割を表す列挙値。ssot / view / sidecar / evidence のいずれか。
- Spec ref (BSL): Chapter 9. Architecture / 6.1
- Depends on (Core): SSOT, View, Sidecar, Evidence
- Status: active

### mapping_kind

- Scope: BSL
- Definition: Mapping の種別を表す列挙値。derive / observe / transfer / transform / compose / manual のいずれか。
- Spec ref (BSL): Chapter 1. Core Concepts / 8.2 Mapping、Chapter 9. Architecture / 6.3
- Depends on (Core): -
- Notes: ssot 更新に関与する経路は transfer / transform / manual のみ許可。derive / observe / compose は ssot 更新の根拠として不可。
- Status: active

### Contract

- Scope: BSL
- Definition: 特定の operation を特定の Space で成立させるために必要な前提の束。最小構成は OperationScope + EvalFrame Φ + RequiredEvidence + EffectDeclaration。
- Spec ref (BSL): Chapter 9. Architecture / 6.3（effect_declaration）、Annex C（Curry–Howard Correspondence）
- Depends on (Core): DEF-Contract、DEF-Φ、AX-8
- Notes: Context は判断の前提条件全体を含み、Contract は Context のうち当該 operation の成立に必要な部分だけを射影したもの（Context ⊃ Contract）。
- Status: active

### effect_declaration

- Scope: BSL
- Definition: operation が外部に要求する依存（effect）を宣言するための Space Metadata 任意キー。
- Spec ref (BSL): Chapter 9. Architecture / 6.3
- Depends on (Core): DEF-Contract
- Notes: OS層は「参照できること」だけを要求し、具体フィールドや payload 形式は Implementation 層で決定してよい。effect_declaration が参照不能な場合は U3 または U4 として扱う。
- Status: active

### undefined_type

- Scope: BSL
- Definition: Checks の判定結果として不成立（undefined）を表す列挙値。U1（cross_space）/ U2（basis_mismatch）/ U3（frame_not_closed）/ U4（evidence_missing）のいずれか。
- Spec ref (BSL): Chapter 9. Architecture / 7.6
- Depends on (Core): DEF-Φ、AX-8、A.4.4（Semantic Discontinuity）
- Notes: U1/U2 が回復不能な場合は Semantic Discontinuity に分類される。U3/U4 は原則として回復可能。
- Status: active

---

## Core Formal Foundations 参照

BSL は以下の Core 公理・定義・補題を前提とする。

### 公理（Axiom）

| 識別子 | 名称 | BSL での影響 |
|--------|------|-------------|
| AX-E1（Evidence の中立性） | Evidence の中立性 | Evidence は因果・意図を表さない。BSL スキーマはこの制約を前提とする |
| AX-E2（Evidence の包含範囲） | Evidence の包含範囲 | 受動的観測＋能動的記録の両方を Reading として扱う |
| AX-6（Sidecar の追記専用性） | Sidecar の追記専用性 | VariationPolicy の append-only 運用の根拠 |
| AX-8（評価フレームの閉包） | 評価フレームの閉包 | 比較・評価には固定された Φ が必須。undefined_type の U3 判定根拠 |

### 定義（Definition）

| 識別子 | 名称 | BSL での影響 |
|--------|------|-------------|
| DEF-E4（Ordering/Sequence 非同一性） | Ordering と Sequence の非同一性 | Ordering スキーマは Sequence スキーマと混同しない |
| DEF-Φ（評価フレームの定義） | 評価フレームの定義 | Φ = (ℐ, 𝒜, 𝒞, 𝒪) の構成要素。eval_frame_ref の参照先 |
| DEF-Contract（Contract の定義） | Contract の定義 | operation 成立に必要な前提の束。effect_declaration を含む |

### 補題（Lemma）

| 識別子 | 名称 | BSL での影響 |
|--------|------|-------------|
| LEM-E3（Trace/Effect 分離） | Evidence と Behavior/Flow の分離規則 | Reading に「行為の結果」を含めない。結果は Behavior/Flow で表現 |
| LEM-4（評価の前提条件） | 評価の前提条件 | 比較には space_id 一致かつ basis_id 一致が必要。Evidence Continuity の根拠 |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | — | 初版。VariationPolicy 関連用語を定義 |
| v0.2 | 2026-01 | Evidence 拡張に対応。Evidence/Reading/Condition/Ordering/trace_type を追加。Core Formal Foundations 参照セクションを追加 |
| v0.2.1 | 2026-01 | BSL Appendix → BSL Annex 参照名変更 |
| v0.3 | 2026-01 | Space 関連用語を追加（Space, Mapping, space_id, basis_id, trace_id, artifact_role, mapping_kind）。LEM-4 を追加 |
| v0.4 | 2026-01 | ToolCall/ToolEffect（Alias）を追加。Reading Notes を更新。References（related policies）セクションを追加 |
| v0.5 | 2026-01 | Contract / effect_declaration / undefined_type を追加。Core Formal Foundations に DEF-Φ / DEF-Contract / AX-8 を追加 |
| v0.5.1 | 2026-03 | 公開前の用語整合パッチを適用 |
