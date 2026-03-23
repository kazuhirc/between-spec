---
version: v0.1
status: draft
updated: 2026-03-24
role: law
sync_note: aligned with gate-completion-table v0.1
---

# Gate completion law

incomplete をこの sequence の終端として扱い、完結条件に加えて「再到達は新しい sequence で行う」を明示する。

根拠は、draft から certified への遷移要件が gate_decision と certify_commit_ref の両方であり、逆遷移は禁止され、修正や再試行は上書きではなく追記または新たな card 発行で表現する、という status rule にある。

また、replay の最小規則は同一 basis、同一 reference bundle、同一 evidence time binding の下で同一 decision を再現できることなので、incomplete からの昇格を同じ sequence の内部で曖昧に継ぎ足すより、新しい sequence として切り直す方が整合的である。

```yaml
gate_completion_law:
  - check が gate_decision=approve を返さない限り certification commit を呼んではならない
  - certification commit は diff_ref, basis_ref, action_trace_ref を伴って成立しなければならない
  - draft_to_certified は gate_decision=approve と certify_commit_ref の両方が揃った場合にのみ許可する
  - check hold と syscall failure を混同してはならない
  - incomplete 状態から certified へ到達するには、新たな sequence を開始し、check と commit を含む完結条件を再度満たさなければならない
```

## 注釈

この sequence は、1 回の select → compare → check → commit を表す単位である。normal commit は commit_status: ok だが certify_commit_ref を伴わないため、failure ではなく incomplete としてこの sequence の終端に入る。draft から certified への遷移には gate_decision=approve と certify_commit_ref の両方が必要であり、incomplete から certified へ到達するには、新たな sequence を開始して完結条件を再度満たさなければならない。

hold は contract が意図した停止であり、reason_code と recovery を伴う。これに対し failure は syscall 自体の不成立であり、undefined_type と diagnostic_reason で返される。したがって incomplete, hold, failure は同じ未完了ではなく、意味的停止、技術的不成立、認証未完了という別々の状態として扱う。
