---
version: v0.1
status: draft
updated: 2026-03-24
role: completion-table
sync_note: aligned with gate-completion-law v0.1
---

# Gate completion table

この表では、check return envelope と commit return envelope を並べて、gate subtype がいつ完結条件を満たすかを明示する。

根拠は 3 点である。gate subtype の outputs は gate_decision と certify_commit_ref を持つこと、draft から certified への遷移要件が「Gate による replay pass と certify_commit_ref の記録」であること、そして commit が diff_ref, basis_ref を伴う採用更新であり、外部作用を伴う場合は approval_ref と authority_ref を要求することである。

| phase | syscall | status field | success branch | required refs on success | failure envelope | gate completion への寄与 |
|---|---|---|---|---|---|---|
| decision | check | check_status | pass または hold | pass では check_result_ref と gate_decision=approve、hold では check_result_ref, triggering_contract_ref, reason_code, failed_fields, failed_refs, recovery_ref | failure では undefined_type, diagnostic_reason | approve のときだけ completion path を開く |
| certification | commit | commit_status | ok | diff_ref, basis_ref, action_trace_ref、certification 条件を満たす場合は certify_commit_ref も必須 | failure では undefined_type, diagnostic_reason | certify_commit_ref を生成し gate outputs を完結させる |

## 横並びスキーマ

```yaml
gate_completion_table:
  check_return:
    check_status: pass | hold | failure
    check_result_ref:
    gate_decision:
    undefined_type:
    diagnostic_reason:
    triggering_contract_ref:
    reason_code:
    failed_fields:
    failed_refs:
    recovery_ref:

  commit_return:
    commit_status: ok | failure
    diff_ref:
    basis_ref:
    action_trace_ref:
    certify_commit_ref:
    undefined_type:
    diagnostic_reason:
```

## Branch matrix

```yaml
gate_completion_matrix:
  - case: check_pass_then_commit_certified
    check_return:
      check_status: pass
      check_result_ref: required
      gate_decision: approve
    commit_return:
      commit_status: ok
      diff_ref: required
      basis_ref: required
      action_trace_ref: required
      certify_commit_ref: required
    gate_completion: complete
    status_transition: draft_to_certified_allowed

  - case: check_pass_then_commit_normal
    check_return:
      check_status: pass
      check_result_ref: required
      gate_decision: approve
    commit_return:
      commit_status: ok
      diff_ref: required
      basis_ref: required
      action_trace_ref: required
      certify_commit_ref:
    gate_completion: incomplete
    status_transition: draft_to_certified_not_yet

  - case: check_hold
    check_return:
      check_status: hold
      check_result_ref: required
      gate_decision: hold
      triggering_contract_ref: required
      reason_code: required
      failed_fields: required
      failed_refs: required
      recovery_ref: required
    commit_return:
    gate_completion: blocked
    status_transition: not_allowed

  - case: check_failure
    check_return:
      check_status: failure
      undefined_type: required
      diagnostic_reason: required
    commit_return:
    gate_completion: blocked
    status_transition: not_allowed

  - case: commit_failure_after_approve
    check_return:
      check_status: pass
      check_result_ref: required
      gate_decision: approve
    commit_return:
      commit_status: failure
      undefined_type: required
      diagnostic_reason: required
    gate_completion: blocked
    status_transition: not_allowed
```

## 読み方

check は gate の判定点である。
pass のときだけ gate_decision=approve を返し、後段の commit に進める。hold は contract が意図した停止であり、reason_code と recovery_ref を伴わなければならない。failure は syscall 自体の不成立で、undefined_type と diagnostic_reason を返す。check_result を evidence に残せない場合は pass でも hold でもなく failure である。

commit は certification を伴う採用更新点である。
正常な commit は少なくとも diff_ref, basis_ref, action_trace_ref を返す。さらに certified 完結を狙う commit では、これに certify_commit_ref が追加される。BSL Shell では、採用判断を伴う commit は diff_ref と basis_ref を伴う必要があり、外部作用を伴う更新では approval_ref と authority_ref を欠いたまま成立させてはならない。したがって certify_commit_ref は、これらの commit evidence bundle を指す参照として置くのが自然である。

## 完結条件

gate subtype の完結条件は、次の 2 つが同時に揃うことである。

1. check_return.gate_decision = approve
2. commit_return.certify_commit_ref が記録されている

この 2 条件が揃ったときだけ、draft から certified への遷移を許可する。これは card system の status transition 規則と一致する。

## 最小規範

gate completion の規範文は [gate-completion-law.md](gate-completion-law.md) に独立して定義している。

## 補足

この表にすると、check は decision syscall、commit は gate completion call だとはっきり見える。

minimal syscall set の核は依然として select, compare, check であるが、gate subtype を certified まで閉じるには commit が準核として必要である。

## JP-EN mini glossary

| JP | EN |
|---|---|
| 完結条件 | completion condition |
| 判定 | decision |
| 採用更新 | commit update |
| 認証付きコミット参照 | certify commit ref |
| 停止 | hold |
| 不成立 | failure |
| 証跡束 | evidence bundle |
