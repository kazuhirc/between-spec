---
version: v0.1
status: draft
updated: 2026-03-24
role: schema
sync_note: aligned with minimal-syscall-set v0.1
---

# Syscall table v0

前提は 3 点である。

select は basis、reference bundle、preconditions を閉じて、どの View をどの Φ で読むかを確定する操作である。compare は contract を読み、中立な Δ だけを返し、decision は返さない。check は stop rule を評価し、check_result を evidence に残せる場合に限って pass または hold を返す。

その前提で、返却スキーマを横並びにすると次の形が最も安定である。

| syscall | 主に読む component | 必須返却 | 条件付き返却 | 返してはいけないもの | 備考 |
|---|---|---|---|---|---|
| select | basis, reference_bundle, preconditions | selected_view_ref, condition_ref, phi_snapshot_ref | undefined_type, diagnostic_reason は failure 時のみ | gate_decision, reason_code | 前提閉包。ここで閉じないものは compare に進めない |
| compare | contract, basis, reference_bundle, preconditions, evidence | delta_ref | undefined_type, diagnostic_reason は failure 時のみ | gate_decision, reason_code, recovery_ref | 判定前の中立差分のみ返す。decision を返さない |
| check | stop_rule, evidence, context from select/compare | check_status | check_result_ref, gate_decision, undefined_type, diagnostic_reason, triggering_contract_ref, reason_code, failed_fields, failed_refs, recovery_ref | delta_ref | stop と failure を分離する。hold では recovery_ref 必須 |

## 1 select return schema

select は、basis と reference bundle と preconditions を読んで、比較可能性の入口を閉じる。BSL Shell では select の ensure が「どの View を、どの Sidecar 条件で、どの Φ で読むか」を確定し、relevant Condition を束ねた condition_ref を定める、と読める構造である。minimal kernel でも basis、reference bundle、preconditions は比較可能性の前提層である。

```yaml
select_return:
  syscall_status: ok | failure

  selected_view_ref:
  condition_ref:
  phi_snapshot_ref:

  undefined_type:
  diagnostic_reason:
```

正常時:

```yaml
select_return:
  syscall_status: ok

  selected_view_ref: <required>
  condition_ref: <required>
  phi_snapshot_ref: <required>

  undefined_type:
  diagnostic_reason:
```

failure 時:

```yaml
select_return:
  syscall_status: failure

  selected_view_ref:
  condition_ref:
  phi_snapshot_ref:

  undefined_type: <required>
  diagnostic_reason: <required>
```

ここで undefined_type は kernel 停止点、diagnostic_reason は具体原因である。basis 不整合は典型的に U2、frame 未閉包は U3、cross space は U1 に落ちる。

phi_snapshot_ref について:

```yaml
phi_snapshot_ref:
  meaning: as_of で束縛された immutable snapshot への参照であり、select 以降の SSOT 更新は当該 sequence に影響しない
  note: approve の有効性は snapshot 自体ではなく、当該 sequence の completion 条件に従う
```

## 2 compare return schema

compare は contract を主に読む。contract セクションは comparison_target, allowed_delta_ref, approval_policy_ref を持ち、minimal kernel はこれを比較対象と許容差の固定層としている。BSL Shell 側でも compare は Reading(kind="delta") を生成するが、判定そのものは分離される。

```yaml
compare_return:
  syscall_status: ok | failure

  delta_ref:

  undefined_type:
  diagnostic_reason:
```

正常時:

```yaml
compare_return:
  syscall_status: ok

  delta_ref: <required>

  undefined_type:
  diagnostic_reason:
```

failure 時:

```yaml
compare_return:
  syscall_status: failure

  delta_ref:

  undefined_type: <required>
  diagnostic_reason: <required>
```

compare は decision を返さない。gate_decision や reason_code を返すと、contract と stop rule の境界が崩れる。

## 3 check return schema

check は stop rule を評価する syscall である。contract 側では stop_rule が reason_code を持ち、発火時は hold record に triggering_contract_ref, reason_code, failed_fields, failed_refs を写像する。さらに recovery は contract 必須で、stop_rule と対でなければ失敗が資産化されない。

```yaml
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
```

pass:

```yaml
check_return:
  check_status: pass

  check_result_ref: <required>
  gate_decision: approve

  undefined_type:
  diagnostic_reason:

  triggering_contract_ref:
  reason_code:
  failed_fields:
  failed_refs:
  recovery_ref:
```

hold:

```yaml
check_return:
  check_status: hold

  check_result_ref: <required>
  gate_decision: hold

  undefined_type:
  diagnostic_reason: <optional>

  triggering_contract_ref: <required>
  reason_code: <required>
  failed_fields: <required>
  failed_refs: <required>
  recovery_ref: <required>
```

failure:

```yaml
check_return:
  check_status: failure

  check_result_ref:
  gate_decision:

  undefined_type: <required>
  diagnostic_reason: <required>

  triggering_contract_ref:
  reason_code:
  failed_fields:
  failed_refs:
  recovery_ref:
```

この切り方だと、hold は contract が意図した停止、failure は syscall 自体の不成立、という区別が崩れない。No gate without a check report と Evidence is append-only から、check_result_ref を evidence に残せない場合は pass でも hold でもなく failure である。

## 4 共通骨格

横に並べて読むと、共通部分と専用部分は次のように整理できる。

```yaml
syscall_common_envelope:
  syscall_name:
  syscall_status:

  primary_ref:

  undefined_type:
  diagnostic_reason:
```

この共通骨格に対して:

- select.primary_ref = selected_view_ref
- compare.primary_ref = delta_ref
- check は単一 primary_ref では足りず、check_status と check_result_ref と hold payload を持つ

check だけが gate 接続点なので、戻り値が厚くなるのは自然である。gate subtype の outputs が gate_decision を持つこととも整合する。

## 5 最小規範

この table を一文で固定するなら、次である。

select は前提閉包の参照を返し、compare は中立差分の参照を返し、check だけが進行停止の判定と hold 導線を返す。

undefined_type と diagnostic_reason は syscall failure 系に属し、reason_code は check hold 系にのみ属する。

check_result_ref を evidence に残せない限り、check は approve も hold も返してはならない。
