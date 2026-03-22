rulesは、v0.1のrequire/ensureと禁止経路を静的に検査するためのルール集である。目的は、比較の成立条件やappend-onlyなど、混線が起きやすい違反を早期に落とすことにある。ルールは実装非依存で、検査対象のスキーマはschemasを参照する。

**modules/rules：最小10ルール案（入力→失敗型→最小修正）**

前提

- undefined_typeは停止点（U1–U4）。diagnostic_reasonに具体の失敗名を入れる。
    
- accept/rejectの判定はpolicy_profile側。ここは「比較可能性の成立」と「採用可能性の最低条件」を機械チェックで止めるだけ。
    
- rulesの出力（check report）は最低限、decision（pass/fail/stop）、u_type（U1–U4）、diagnostic_reason、missing（参照束の不足）、pointers（再検査に必要なref）を含む。
    
- rulesは比較可能性の成立条件を落とすための検査であり、accept/rejectはpolicy_profileに委譲する。

設計原則（v0.1 からの導出）

以下は v0.1 の公理・補題から導出される運用原則であり、v0.2 modules のルール群が前提とする判断基準を要約する。
Ref: AX-8, LEM-4, LEM-5

- Rule: Interpretation is explicit.
- Rule: Comparability requires declared references (artifact_role / space_id / basis_id / eval_frame_ref).
- Rule: If any required reference is missing, hold the gate and request remediation (non-comparable).
- NOTE: No single metric governs all spaces; comparability is local to a declared frame (Φ and basis).
- NOTE: Promotion to SSOT is gated; ambiguity may remain in view/evidence by policy.

|   # | ルール（機械チェック）                                    | 入力（最低限）                               | 失敗型（停止点→理由）                        | 最小修正（機械が返す指示）                                                                   |
| --: | ---------------------------------------------- | ------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------- |
|   1 | eval_frame_refの参照可能性                           | event.eval_frame_ref                  | U1 → EvalFrameRefMissing           | eval_frame_refを付与（SSOT上のΦスナップショットIDに束縛）。参照先が無い場合はΦを先に作成。                        |
|   2 | comparison_scopeの確定（比較単位）                      | event.comparison_scope                | U1 → ComparisonScopeMissing        | comparison_scopeを明示（例：module/api-surface/cli）。scopeが閉じない場合は対象を縮小。               |
|   3 | artifact_roleの明示（ssot/view/sidecar/evidence）   | artifact.role                         | U2 → ArtifactRoleMissing           | roleを付与。LLM出力は原則view。採用済みはssot。条件束はsidecar。観測結果はevidence。                       |
|   4 | allowed_edge_kind違反（read/write/derive/compare） | edge.kind, contract.allowed_edges     | U2 → EdgeKindNotAllowed            | Step Contractにallowed_edgesを追加するか、操作をderiveへ落とす（write/compareを禁止するなら候補生成だけに戻す）。 |
|   5 | viewからSSOTへの直接write禁止                          | edge: view → ssot (write)             | U2 → DirectWriteFromViewProhibited | viewをevidenceへ接地（テスト/解析/ログ）し、Decision Unit（approved）を経由してcommitへ進める。            |
|   6 | Space跨ぎ直接比較禁止（射影必須）                            | compare: space_a, space_b, mapping_id | U3 → DirectComparisonAcrossSpaces  | mapping_id（射影）を指定し、比較対象を同一Basisへ正規化してからcompareする。                               |
|   7 | Conditionの固定（sidecar）欠落                        | sidecar.condition_ref                 | U1 → ConditionMissing              | condition_refを付与（依存版/環境変数/入力データ/seed/TZ等）。未確定なら比較不能で停止。                         |
|   8 | Orderingの固定（sidecar）欠落                         | sidecar.ordering_ref                  | U3 → OrderingMissing               | ordering_refを付与（型→解析→単体→統合→E2E…）。順序が未合意なら比較不能で停止。                               |
|   9 | evidenceのtrace_id必須（再現・回帰の戻り点）                 | evidence.trace_id                     | U4 → EvidenceTraceMissing          | trace_idを付与（CI run id / build id / log span id等）。無いevidenceは採用根拠に使わない。          |
|  10 | evidenceのappend-only（改竄/上書き禁止）                 | evidence.event_id, evidence.prev_ref  | U4 → EvidenceNotAppendOnly         | event_idの単調付与とprev_refで鎖を作る。過去evidenceの上書きを禁止し、新規イベントとして追記する。                   |
