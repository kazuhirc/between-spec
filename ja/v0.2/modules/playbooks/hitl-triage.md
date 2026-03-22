**HITL Triage**
（止める→揃える→再開：v0.2 playbook）

この文書は、BSL Shellが返す失敗（undefined_type + diagnostic_reason）を、人間が一貫した手順で処理するための運用レシピである。
目的は「議論（責任論）」を「型エラー（比較不能・不成立）」へ変換し、最小修正で再試行できる状態へ戻すことである。
判断の焦点は「真偽」ではなく「成立条件の不足」であり、不足がある場合は比較不能として停止する。
v0.2では、停止点（U1–U4）を増やさず、理由（diagnostic_reason）と回復（recovery）を厚くする。

---

**0. 入力（Triageに必要な最低情報）**

次のセットが揃っていない場合、まず収集に戻る（比較不能で停止する）。
- operation_kind（例：select / compare / check / commit）
- undefined_type（U1–U4）
- diagnostic_reason（具体の失敗）
- 対象ArtifactRef（ssot/view/sidecar/evidence）とspace_id/basis_id（分かる範囲）
- 直近のEvidence（trace_id / check_result / action_trace の有無が分かる範囲）

任意（あれば回復が速い）
- non_comparable_reason（missing/mismatch/leak/disconnection/invalid）
- boundary_kind（interface/authority/execution/evidence）
- required_evidence_tier / observed_evidence_tier（t0/t1/t2）

---

**1. 1分で切る：まず「止める」**

最初に決めるのは「続行」ではなく「停止理由」である。
U1–U4の停止点は、最初に落ちたゲートとして尊重し、後から別の停止点へ“付け替えない”。
複数の問題が見えても、回復は1つずつ行う（最小修正の原則）。

停止点の優先度（固定）
- U1 → U2 → U3 → U4

---

**2. 3問チェック（停止時の最小説明ユニット）**

停止したら、次の3問に短く答える。答えが出ない問いは、そこが不足である。
1) SSOTはどれか。今回の操作で参照するSSOTの束は何か。
2) Basis/Sidecar/Φの閉包はどこまで確定しているか（I/A/C/O、condition_ref、ordering_ref）。
3) 再現可能性を保証するEvidence鎖は何か（trace_id、ordering、check_result、as-of束縛）。

3問の答えを埋められない場合は、比較不能として停止し、欠落参照をblocking_missingとして列挙する。

---

**3. U1–U4別の処理（原因→最小修正→再試行）**

**U1: reference set不足 / cross-space（参照束が揃っていない）**
典型
- open不足、artifact_id/role/space_id不足
- Φが解決不能（eval_frame_ref または I/A/C/O参照が欠ける）
- ssotとsidecarのspaceが揃っていない
- view/sidecarにbasis_idがない（未取得で参照不能）

最小修正
- openを先に行い、参照束（ArtifactRef）を確定する
- role条件付きキー（basis_id / trace_id）を埋める
- Φ参照（eval_frame_ref か I/A/C/O参照）を揃える
- space_idを揃えるか、comparison_scopeを縮小して同一spaceに落とす

再試行
- select（参照束が揃っていることを確認）→ compareへ戻る

Related rules: #1 eval_frame_ref, #2 comparison_scope, #7 condition_ref

---

**U2: scope/mapping/basis mismatch（比較単位や写像が一致していない）**
典型
- basis_id不一致、Φ不一致
- mapping_kind不正、射影（mapping）が明示されていない
- artifact_roleがスコープ外（OperationScope違反）

最小修正
- 比較単位（comparison_scope）を明示し、揃う単位へ縮小する
- basis_id/Φを揃える（揃えられない場合は比較不能にする）
- mapping_kindを明示し、compareの前にderive（射影）で同型に落とす
- OperationScopeの宣言と実体（artifact_role）を一致させる

再試行
- derive（同型化）→ compare（Φ/basis一致）→ check へ進む

Related rules: #3 artifact_role, #4 allowed_edge_kind, #5 direct write from view prohibited

---

**U3: frame not closed / policy violation（閉包違反・禁止則）**
典型
- 外部依存があるのにeffect_declaration_refがない
- approval_ref/authority_refが必要な更新で欠落している
- ViewをEvidenceとして扱う、ViewからSSOTへ逆流する
- direct comparison prohibition（Space跨ぎの見た目比較）をした

最小修正
- effect_declaration_refを宣言し、依存境界（Execution/Evidence）を閉じる
- approval_ref/authority_refを揃えるか、外部作用を持つ更新を諦めて止める
- Viewは候補に戻し、採用はDecision Unit（approved）経由にする
- 直接比較をやめ、必ず射影（mapping）を明示してから比較する

再試行
- adopt（必要なら）→ commit（根拠参照束）→ re-derive → compare

Related rules: #6 direct comparison across spaces, #8 ordering_ref

---

**U4: evidence chain broken（証跡連鎖が成立しない）**
典型
- evidenceにtrace_idがない（EvidenceTraceMissing）
- append-onlyが破れている、orderingが不連続
- check_resultが書けない
- as-of束縛がなく再現できない

最小修正
- まずEvidence鎖を作り直す（trace_id付与、orderingの単調性回復）
- check_result（pass/fail対称）を必ず追記できる状態に戻す
- どうしても復旧できない場合、比較不能として停止し、再収集へ戻す

再試行
- capture/order/check を先に成立させ、その後に compare/commit を行う

Related rules: #9 evidence.trace_id, #10 evidence append-only (event_id + prev_ref)

---

**4. non-comparability（比較不能）を正規状態として返す**

比較不能は「失敗」ではなく「成立条件不足」である。
次の粗分類を使い、diagnostic_reasonは具体を返す。

- missing: 参照や証跡が不足している
- mismatch: 基準や写像が一致しない
- leak: 条件や境界外情報が混入している
- disconnection: 鎖（ordering/trace/as-of）が断絶している
- invalid: 禁止則違反、権限不備、スコープ違反

運用ルール
- missing/mismatch/disconnection は、まずblocking_missing（不足参照）を列挙して止める
- leak/invalid は、まず禁止則の撤回（手順のやり直し）として止める

---

**5. boundary taxonomy（どの境界が破れたか）**

境界は「原因の切り分け」であり、責任の押し付けではない。
diagnostic_reasonに加えて、可能なら boundary_kind をタグとして付与する。

- interface: 型・単位・座標系・スキーマ・basisの不一致
- authority: approval/authority/責任の境界が閉じていない
- execution: 実行環境・外部依存（effect）の境界が閉じていない
- evidence: 証跡と派生の混同、trace/ordering/as-ofの欠落

---

**6. Evidence tier（コストと強度を揃える最小運用）**

Tierは「その場の正しさ」ではなく「再現可能性の強度」を表す。
required_evidence_tier を満たさない場合は、比較不能として止める。

推奨（最小）
- t0: trace_id + action_trace + check_result（最低限の戻り点）
- t1: t0 + ordering（単調性）+ as-of束縛（再実行条件の固定）
- t2: t1 + 追加の検証束（静的解析/CIログ/差分説明）

注記
- tierは運用の“買い物”であり、最初からt2を要求しない
- ただし採用（commit）を伴う操作は、t0未満では禁止する

---

**7. 迷ったときの基本方針（HITLの合言葉）**

- 分からないなら止める（比較不能として返す）
- 直さないで増やさない（最小修正で戻す）
- Viewは候補、採用はDecision Unit（approved）
- 直接比較しない。射影してから比較する
- 証跡鎖が無いなら、まず鎖を作る

---

**8. 付録：最小修正（Recovery）カードの形**

運用で使う最小カードは次の4行でよい。
- Symptom: （U? + diagnostic_reason）
- Blocking: （不足参照 or 禁止則）
- Minimal fix: （追加するref / やり直す手順）
- Retry: （次に叩くoperation_kind）
