# BSL Annex C: Curry–Howard Correspondence for Between (informative)

**Version:** v0.1

---

## Purpose

本 Annex は、Between の操作体系と Curry–Howard 対応（命題＝型、証明＝プログラム）の関係を整理し、
Between の形式的基盤を型理論の観点から解釈するための参考資料である。

本 Annex は informative であり、normative な定義は Core axioms および BSL 本文を参照のこと。

---

## C.1 Correspondence (Curry–Howard × Between)

| Curry–Howard | Between | 説明 |
|--------------|---------|------|
| Proposition（命題） | Contract | operation 成立に必要な前提の束 |
| Proof（証明） | Evidence | trace_id と evidence_chain（append-only）によって構成される正当化 |
| Program（計算） | Operation / Mapping | mapping_kind（derive / observe / transfer / transform / compose / manual）と、その適用手順 |

### Contract の最小構成

Contract は、特定の operation を特定の Space で成立させるために必要な前提の束である。
最小構成は次の通りである。

```
Contract = OperationScope + EvalFrame Φ + RequiredEvidence + EffectDeclaration
```

| 構成要素 | 役割 |
|---------|------|
| OperationScope | 操作範囲の宣言（許可される artifact_role、operation_kind 等） |
| Φ | 評価フレーム（DEF-Φ 参照） |
| RequiredEvidence | 成立に必要な Evidence（trace_id 等） |
| EffectDeclaration | 外部依存の宣言（外部ツール、辞書、モデル等への参照） |

> 参照：Core DEF-Contract

---

## C.2 Judgement form on 3×3 (Space × Layer)

Between の操作は、Space と Layer 上の判断として表す。

$$
(S, L) \vdash op : a \to b \; [\varepsilon] \; \{E\}
$$

| 記号 | 意味 | BSL での対応 |
|------|------|-------------|
| $S$ | space_id | Space Metadata の space_id |
| $L$ | layer | OS / Implementation / Application |
| $op$ | mapping_kind（または操作名） | derive / observe / transfer / transform / compose / manual |
| $a \to b$ | 入力の意味構造 → 出力の意味構造 | operation の型シグネチャ |
| $[\varepsilon]$ | effect_declaration | 外部依存の宣言（BSL_9 6.3） |
| $\{E\}$ | RequiredEvidence | trace_id, eval_frame_ref 等 |

### Key mapping (informative)

| BSL キー | 判断形式 |
|----------|----------|
| space_id | $S$ |
| layer | $L$ |
| mapping_kind | $op$ |
| effect_declaration | $[\varepsilon]$ |
| trace_id / eval_frame_ref | $\{E\}$ |

---

## C.3 Undefined as first-class result

不成立（undefined）は、例外ではなく結果型として扱う。
これは Curry–Howard 対応において「証明が立たない」（項が構成できない）ことに相当する。

### undefined_type (normative)

| 値 | 名称 | 説明 |
|----|------|------|
| U1 | cross_space | space_id が不一致で、直接比較が禁止される |
| U2 | basis_mismatch | basis_id が不一致で、比較の座標系が揃っていない |
| U3 | frame_not_closed | eval_frame_ref が欠落または参照不能で、Φ が閉じていない |
| U4 | evidence_missing | trace_id が欠落、または必要な evidence_chain が欠落し、採用・再現ができない |

### Interpretation (informative)

- U* は「証明が立たない」ことに対応する。
- 不成立は境界が機能しているシグナルであり、停止・分類・証跡化の起点とする。
- U1/U2 が回復不能な場合、Core A.4.4 の Semantic Discontinuity に分類される。
- U3/U4 は原則として回復可能（欠落要素を補完可能）であり、Discontinuity とは区別される。

> 参照：BSL_9 7.6（Checks Result Types）、Core A.4.4（Semantic Discontinuity）

---

## C.4 Design Rationale (informative)

### なぜ Curry–Howard 対応を導入するか

1. **形式的基盤の明確化**: Between の操作体系を型理論の言葉で説明できるようになり、形式検証への道が開ける。

2. **不成立の第一級化**: 例外やエラーではなく、型として不成立を扱うことで、境界条件の検査が自然に設計に組み込まれる。

3. **Contract の明示化**: operation の成立条件を Contract として束ねることで、前提の欠落や不整合を早期に検出できる。

### 制約と限界

- 本対応は Between の操作を解釈するための参考枠組みであり、Between 自体が依存型言語や証明支援系を要求するものではない。
- 実装は Contract の形式検査を必須としない。ただし、undefined_type の分類は normative として扱う。

---

## References

| 参照先 | 関係 |
|--------|------|
| Core DEF-Φ | 評価フレームの定義 |
| Core DEF-Contract | Contract の定義 |
| Core AX-8 | 評価フレームの閉包 |
| BSL_9 6.3 | effect_declaration |
| BSL_9 7.6 | undefined_type |
| Core A.4.4 | Semantic Discontinuity |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | 2026-01 | 初版。Curry–Howard 対応表、judgement 形式、undefined_type を定義 |
