---
version: v0.1
status: draft
updated: 2026-03-24
role: mapping
depends_on:
  - kernel-memo
---

# BSL Shell operation × kernel component 対応表

この対応表は、BSL Shell の operation を「kernel component をどう読むか」という観点で整理したものである。

前提として、kernel 側の最小語彙は basis, reference bundle, preconditions, contract, stop rule, recovery, evidence, outputs の 8 つであり、比較可能性は basis 解決、bundle 閉包、preconditions 充足、evidence 時点束縛で成立する。

また BSL Shell v0.1.39 は、実装ではなく「操作の意味と契約」を定義し、open, select, derive, compare, check などを require / ensure で固定している。

| operation | 主に読む kernel component | 補助的に触る component | 役割 |
|---|---|---|---|
| open | evidence | basis | 参照導入の成立を action trace として残す |
| select | basis, reference bundle, preconditions | evidence | どの View をどの Φ で読むかを確定する |
| derive | basis, reference bundle | evidence | SSOT と sidecar から決定的に View を再計算する |
| perturb | preconditions | evidence | 比較条件の差を Condition として明示する |
| compare | contract | basis, reference bundle, preconditions, evidence | 判定前の中立な Δ を作る |
| check | stop rule | basis, reference bundle, preconditions, evidence, outputs | 継続可否を判定し pass/fail を記録する |
| capture | evidence | preconditions | Reading と Condition を append-only で追記する |
| order | evidence | reference bundle | ordering_ref を固定して再現性を閉じる |
| commit | outputs | basis, evidence | diff_ref, basis_ref を伴う採用更新を記録する |
| recovery | recovery | stop rule | stop 後の最小修正と再試行条件を指す |

## 1 open

open は kernel の判断条件をまだ組み立てない。

ここで主に触るのは evidence である。ArtifactRef が最小メタデータを満たすことを require とし、成功時は Reading(kind="action_trace") を追加する。つまり open は contract evaluation ではなく、参照導入ログである。

ただし role が view や sidecar の場合は basis_id、evidence の場合は trace_id が条件付き必須なので、basis や evidence の入口検査にも接している。

## 2 select

select は kernel 接合面の中心である。

主に読むのは basis, reference bundle, preconditions である。require として、同一 space_id、必要な basis_id、解決可能な EvalFrameRef、必要なら EffectDeclarationRef を要求し、ensure として「どの View を、どの Sidecar 条件で、どの Φ で読むか」を確定する。さらに relevant Condition を束ねた condition_ref を確定する。

したがって select は、Pre-execution layer を閉じる operation と見てよい。kernel memo で言う basis 解決、reference bundle 閉包、preconditions 充足へ最も直接に対応する。

## 3 derive

derive は contract を直接判定しない。

主に読むのは basis と reference bundle である。入力は SSOT と sidecar で、依存方向を破らず、同一入力なら同一出力でなければならないとされている。出力 View は basis_id を持つ。

つまり derive は、compare や check の前に比較対象を決定的に生成する operation であり、kernel 側では「basis に拘束された projection 生成」と読める。

derive は minimal syscall set の核には入らないが、select が basis, reference bundle, preconditions を閉じた後に、その条件に従って View を決定的に生成する supporting call である。select は「何を読むか」を確定し、derive は「その条件での実体」を返す。したがって executor が compare に渡す比較対象は derive の出力であり、select の返却値だけでは比較対象の実体は得られない。

## 4 perturb

perturb は比較条件を動かすが、SSOT は動かさない。

主に preconditions と evidence に接しており、摂動条件を Condition として宣言し、比較に使う条件差を明示する。

したがって perturb は、kernel の preconditions を書き換えるというより、「条件差を evidence 側へ明示して次の select / compare に渡す」operation である。

## 5 compare

compare は contract の本体に最も強く接する。

require として同一 space_id、一致した Φ、一致した basis_id を要求し、ensure として「判定ではなく差分の事実」を Reading(kind="delta") として残す。

ここで読む kernel component は contract である。具体的には comparison_target と allowed_delta_ref が compare 面を構成し、basis, reference bundle, preconditions, evidence は compare を許す前提として作用する。kernel memo でも contract は比較対象、許容差参照、承認方針を固定する component とされており、この対応は明確である。

## 6 check

check は stop rule に最も直接に接する operation である。

require として必要参照が揃っていることを求め、ensure として undefined_type と diagnostic_reason の 2 段で返し、pass/fail を Reading(kind="check_result") として append しなければならない、とされている。check_result を追記できない場合は U4 である。

kernel 側では、check は stop rule を評価し、必要なら hold へ落とす前段である。minimal kernel でも stop rule は contract 必須で、stop 発火時は reason_code を hold に写像する。

このため check は Governance layer の主 operation と見なせる。

## 7 capture

capture は evidence を増やす operation である。

trace_id と append-only を require とし、Reading と Condition を追記する。Condition は compare 前に relevant set として condition_ref に束ねられ、次の select に受け渡される。

つまり capture は kernel の判定面そのものではなく、preconditions と evidence を後続 operation のために整える役である。

## 8 order

order は evidence のうち Ordering を固定する operation である。

Ordering が単調増加でチェイン整合することを require とし、以後の比較が ordering_ref を含む Φ でのみ再現可能になることを ensure とする。

kernel memo では evidence の時点束縛が replay 条件の一部であり、same basis × same reference bundle × same evidence time binding が replay invariance を支える。order はその evidence 側の固定点である。

## 9 commit

commit は card contract の compare や check とは少し役割が異なるが、kernel 近傍である。

require として diff_ref、外部作用がある場合の approval_ref と authority_ref、採用判断を伴う場合の basis_ref を求め、View から SSOT への直接更新を禁止する。

したがって commit が主に触るのは outputs と evidence である。kernel の 8 component には approval_ref や authority_ref はまだ昇格していないので、commit は minimal kernel の外縁にある operation と見た方が正確である。つまり「kernel に完全収まる operation」ではなく、「kernel 制約を守りながら採用更新を行う operation」である。

## 10 recovery

BSL Shell v0.1.39 には recovery という独立 operation はないが、kernel 側には component として recovery がある。

役割は、stop 後に minimal_fix、retry_conditions、triage_playbook_ref を与えることである。stop_rule と対にならないと失敗が資産化されない、と minimal kernel は明記している。

したがって operation 対応表では、recovery は「check や hold emit の後に参照される導線」として置くのがよい。

## 圧縮した読み

この表を一段圧縮すると、次の対応になる。

- Pre-execution: open → select → derive — ここで basis, reference bundle, preconditions, evidence の前半を閉じる
- Governance: compare → check — ここで contract と stop rule を評価する
- Post-execution: capture → order → commit — ここで evidence と outputs を固定する
- Repair path: recovery — ここで stop 後の再試行条件を与える

## 暫定結論

いまの段階では、最も重要な接合はこう言える。

- select は basis + reference bundle + preconditions を閉じる
- compare は contract を読む
- check は stop rule を評価する
- capture, order, commit は evidence と outputs を固定する

この 4 行まで落とせれば、kernel と executor の接合面としてはかなり見通しが立つ。
