# BSL Shell Dialogue Annex v0.2
## Open Loop Pragmatics and Closed Loop Pragmatics

## Changelog

- v0.2-draft: initial draft (open/closed loop pragmatics, ask/assert/repair/revise, pragmatic state)
- v0.2-patch-1: repair/revise boundary tightened to add-only vs supersede. agreed_condition_ref defined as candidate set for select. incomparable_reason_ref designated as normative term. closure check removed from operation_kind, repositioned as select pre-condition.
- v0.2-patch-2: §5.6 repair example aligned with add-only constraint. §6 Mismatch corrected to reference agreed_condition_ref. §5.8 re-entry rules split for readability. §5.4 terminology note added (candidate set / relevant set).
- v0.2-patch-3: §2 v0.1 compatibility guarantees restored. §3 dialogue_state_ref に turn_ref (optional) を復元。§3 reference_binding_policy_ref の最小 3 項目を復元。§3 repair_priority_ref に診断優先順位の典型列を追記。§4 Representation note (informative) を復元。

## 1 Purpose

本 Annex の目的は、比較成立前の未確定状態を扱うための語彙と操作を与えることである。

v0.1 Shell は、`select` `check` `compare` `commit` を中心に、比較可能性と停止条件を実装非依存に定義する。これに対して本 Annex は、比較に入る前の対話的な前処理、すなわち「何が未確定か」「何を誰に確認すべきか」「どの前提を暫定的に束ねたか」を扱う。ここで扱うのは真偽の判定ではなく、比較へ進むための参照束縛と前提束の整流である。

本 Annex は、open loop pragmatics と closed loop pragmatics を区別する。open loop pragmatics は未確定参照と未確定前提を管理する層であり、closed loop pragmatics は停止理由を固定し、最小修正で比較成立へ戻す層である。両者の最終目的は、v0.1 の `select` / `check` へ戻って比較を前に進めることにある。

## 2 Scope

本 Annex の射程は、対話に由来する派生状態の保持、未解決問いの管理、暫定的な参照束縛、停止理由に基づく回復手順の整理に限る。

本 Annex は、`dialogue_state_ref`、`binding_set`、`open_questions`、`agreed_condition_ref`、`adopted_evidence_refs` を扱う。また、operation として `ask` `assert` `repair` `revise` を定義する。これらは v0.1 の外側で働く拡張であり、SSOT 更新の代用ではない。SSOT 更新が必要な場合は、常に v0.1 の `commit` 手続へ戻る。

本 Annex は、v0.1 の停止点を増やさない。不成立は常に v0.1 の二段返し、すなわち第一成分 `undefined_type` = U1–U4 と、第二成分 `diagnostic_reason` により表現する。対話側の失敗は型エラーを置き換えず、「何が未確定か」という観測として Sidecar に残し、次の `ask` または `repair` の入力に用いる。

本 Annex の非射程は、全文会話ログの標準化、自然言語理解の内部実装、対話UI、発話生成器、責任帰属の社会的判断、外部承認ワークフローそのものの定義である。本 Annex は比較可能性の前段を整えるが、意味論的な核や運用制度全体を置き換えない。

本 Annex は v0.1 に追加するだけであり、v0.1 の語彙・契約・型エラー・禁止経路の削除や意味変更を行わない。v0.1 の `operation_kind` は `read` `derive` `compare` `write` に限定し、本 Annex の operation はその外側に置く。対話上の合意や主張を v0.1 の `commit` と呼ばない。

NOTE: 本 Annex は `incomparable_reason_ref` を規範語とする。既存 playbook の `non_comparable_reason` / `non_comparable_reason_ref` は同一概念の運用表示語であり、本 Annex の規範語と対立しない。将来の用語統一は Glossary 側で管理する。

## 3 Glossary

本節は、v0.1 Shell が予約語として置いた dialogue 拡張語と、本 Annex 内でのみ用いる局所語を定義する。BSL 本体の既存語は再定義しない。

### open loop pragmatics

比較成立前に、未確定参照と未確定前提を管理する操作層である。
この層の目的は、結論を先に出すことではなく、何が未確定かを明示し、比較へ進むための参照集合を揃えることにある。open loop pragmatics は SSOT を更新しない。保持先は Sidecar 上の `dialogue_state_ref` である。

### closed loop pragmatics

停止理由を固定し、最小修正で比較成立へ戻す操作層である。
この層の目的は、失敗を曖昧な会話のまま流さず、`undefined_type` と `diagnostic_reason` により停止点を明示し、再試行可能な状態へ戻すことにある。closed loop pragmatics は v0.1 の `select` `check` `compare` と整合しなければならない。

### pragmatic state

対話由来の派生状態であり、`dialogue_state_ref` により参照される。
最小形は `space_id` `basis_id` `eval_frame_ref` `binding_set` `open_questions` `agreed_condition_ref` `adopted_evidence_refs` からなる。これは SSOT ではなく、必要なら Evidence と Sidecar から再構成可能でなければならない。

### pragmatic closure

`binding_set` と `agreed_condition_ref` が十分に揃い、`select` に渡せる状態である。
この状態では、比較の前提となる Space Basis Φ Condition の束が局所的に閉じている。pragmatic closure は compare 成立そのものではなく、compare へ進める入口条件である。pragmatic closure の判定は `select` の require を満たすかどうかの前段確認であり、独立した operation_kind ではない。

### pragmatic failure

比較へ進む前に、未確定参照、前提不足、参照競合、禁止経路接触、証跡不足などが存在する状態である。
pragmatic failure は会話の失敗ではなく、成立条件不足の観測である。この状態は `undefined_type` と `diagnostic_reason` に写像され、Sidecar に記録される。

### pragmatic recovery

`ask` `assert` `repair` `revise` を用いて pragmatic closure を回復する手順である。
recovery は最小修正を原則とし、一度に複数の停止点を混ぜない。回復後は必ず `select` `check` へ戻り、比較可能性を再評価する。

### dialogue_state_ref

対話により確定した参照束縛と前提を保持する派生状態への参照である。
実装は内部表現に依存してよいが、少なくとも `space_id` `basis_id` `eval_frame_ref` `binding_set` `open_questions` `agreed_condition_ref` `adopted_evidence_refs` を解決可能でなければならない。`turn_ref`（optional）は対話ターンの参照であり、並列対話の分離に用いる。並列対話を扱う実装では解決可能でなければならない。全文会話ログを固定する必要はない。

### binding_set

談話内の識別子を ArtifactRef または SidecarRef に束縛した集合である。
役割は、曖昧参照を局所的に安定化し、v0.1 の参照解決へ橋渡しすることにある。Mismatch が生じた場合は、`binding_set` の再束ねが recovery の対象となる。

### open_questions

比較へ進む前に埋めるべき未解決の問いの集合である。
`ask` は不足参照や不足条件をここに追加する。open_questions 自体は結論を持たず、不足の所在を可視化するだけである。

### agreed_condition_ref

対話により一旦合意された前提の束への参照である。
これは v0.1 の `condition_ref` の候補束であり、compare 前の仮固定面を構成する。`select` が候補束から relevant set を確定し、v0.1 の `condition_ref` に正規化する。`agreed_condition_ref` を `condition_ref` と直接同一視してはならない。`revise` が必要な場合は、旧前提を削除せず、supersede relation を伴う追記として改訂する。

### adopted_evidence_refs

今回の対話で採用した Evidence 参照の集合である。
役割は、何を根拠として前提を束ねたかを後から再確認可能にすることにある。Evidence は append-only を守り、再現可能性を損なってはならない。

### ask

不足している参照または条件を明示し、`open_questions` に追加する操作である。
`ask` は SSOT や View を更新しない。更新が必要な場合は v0.1 の `commit` 手続へ戻る。

### assert

対話上の主張または暫定合意を Sidecar に追加する操作である。
`assert` は SSOT 更新ではない。compare や commit の代用として用いてはならない。

### repair

`check` が返した失敗に対して、最小修正を適用し、参照束と前提束を局所的に束ね直す操作である。
`repair` は判定を混入せず、回復だけを担当する。`repair` は対話側の状態に対して add-only であり、少なくとも `binding_set` と `agreed_condition_ref` への要素追加を許す。既存要素の無効化や差し替えを伴う場合は `revise` を用いる。

### revise

既存の束縛や合意前提を改訂する操作である。
`revise` は append-only を破らず、旧状態を削除しない。改訂は supersede relation（どの旧要素をどの新要素が置き換えるか）を Sidecar 上で明示する追記として表現する。supersede relation は Sidecar の記録から回復可能でなければならない（たとえば action_trace または Condition-linked reference として残す）。`repair` が add-only の局所補完なら、`revise` は supersede を伴う前提集合の再編である。

### reference_binding_policy_ref

曖昧参照をどの規則で束縛するかを定める参照である。
少なくとも次の 3 項目を含まなければならない。

- 参照の優先順位（v0.1 の参照優先順位に従うか、追加の上書き規則を持つか）
- 束縛の安定性（ターン内限定か、セッション内固定か）
- 束縛の可視化（何が束縛されたかを action_trace で残す）

open loop pragmatics の安定性を支える contract である。

### repair_priority_ref

失敗から回復する際に、どの不足から先に埋めるかを定める参照である。
少なくとも v0.1 の診断優先順位と矛盾してはならない。典型的には、評価フレーム、Space/Basis、Evidence 連鎖、役割/条件、依存/手続、決定性の順に不足を切り分ける。
これは closed loop pragmatics の回復順序を固定する contract である。具体的な適用順は modules / playbooks 側で定義してよい。

### incomparable_reason_ref

不成立理由の型への参照である。
第一成分として `undefined_type` を持ち、第二成分として `diagnostic_reason` を持つ。追加の詳細は Sidecar に置き、判定や結論を混入しない。
NOTE: 規範語は `incomparable_reason_ref` とする。運用文書（triage-playbook 等）で用いられる `non_comparable_reason` は表示ラベルであり、規範上の参照名ではない。両者は同一系列の概念を指す。

### blocking_missing

比較成立に必要だが、現時点で欠けている必須参照の列挙である。
停止時の 3 問チェックに答えられない場合は、比較不能として停止し、`blocking_missing` を返す。これは open loop へ戻すための入力である。

### stop reason

続行ではなく、先に固定する停止理由である。
停止点は最初に落ちたものを尊重し、後から別の停止点に付け替えない。これは closed loop pragmatics の規律である。

### minimal correction

一度に一つだけ行う最小修正である。
複数の問題が見えても、回復は一つずつ行う。これは recovery の再現性を保つための粒度制約である。

## 4 Model

本 Annex は、対話を SSOT ではなく派生状態として扱う。モデルの中心は `dialogue_state_ref` であり、これは比較成立前の未確定性を局所化するための Sidecar 参照である。

モデルは二層からなる。第一層は open loop pragmatics であり、`ask` と `assert` を通じて未解決問い、参照束縛、暫定前提、採用 Evidence を蓄積する。ここでは結論を出さず、比較に必要な束を揃える。第二層は closed loop pragmatics であり、`select` / `check` によって比較成立可否を評価し、失敗した場合には `repair` または `revise` により最小修正を行う。

open から closed への遷移条件は pragmatic closure である。少なくとも `space_id` `basis_id` `eval_frame_ref` `binding_set` `agreed_condition_ref` が局所的に揃い、`select` に渡せる状態でなければならない。pragmatic closure の判定は `select` の require を満たすかどうかの前段確認であり、独立した operation_kind ではない。逆に、3 問チェックに答えられない場合、または停止理由が U1–U4 のいずれかで確定した場合は、比較を前に進めず、open loop 側へ戻る。

本モデルは v0.1 の型エラーを置き換えない。対話由来の失敗は、v0.1 の停止点を観測し、次の確認や修復に使うための外側のループである。したがって、本 Annex は意味論的核を変更せず、比較成立前段の整流器として働く。

NOTE (informative): DialogueState は、Sidecar 上の `Reading(kind="dialogue_state")` として表現してよい。ただし、対話内容の全文を規範として固定する必要はなく、比較成立に必要な参照束縛と前提だけを残す。

## 5 Operations

### 5.1 Ask

`ask` は、不足している参照または条件を明示し、`open_questions` に追加する。
入力は未確定参照、未確定条件、または `check` が返した失敗である。出力は更新された `open_questions` と `dialogue_state_ref` である。`ask` は SSOT や View を更新しない。

### 5.2 Assert

`assert` は、対話上の主張または暫定合意を Sidecar に追加する。
入力は束縛候補、条件候補、根拠候補である。出力は更新された `binding_set`、`agreed_condition_ref`、`adopted_evidence_refs` である。`assert` は commit ではなく、compare の代用でもない。

### 5.3 Closure Check（select の前段条件）

closure check は、open loop で集めた状態が `select` の require を満たすかを確認する前段判定である。独立した operation_kind ではなく、Sidecar への operation 記録対象ではない。
少なくとも `space_id` `basis_id` `eval_frame_ref` `binding_set` `agreed_condition_ref` が局所的に揃っていることを確認する。揃っていない場合は `blocking_missing` を返し、open loop に留まる。

### 5.4 Select

closure が成立した状態を v0.1 の `select` へ渡し、view / basis / condition を確定する。
ここで `select` は `agreed_condition_ref`（候補束）から relevant set を確定し、v0.1 の `condition_ref` に正規化する。`binding_set` は ArtifactRef / SidecarRef の解決入力として使われる。

NOTE: 本 Annex では、`agreed_condition_ref` を候補束（candidate set）、`condition_ref` を `select` により確定された relevant set と呼ぶ。

### 5.5 Check

`check` は、v0.1 の規則に従って成立可否を評価する。
通過した場合は compare へ進む。失敗した場合は、第一成分 `undefined_type` = U1–U4 と第二成分 `diagnostic_reason` により停止理由を確定し、その結果を Sidecar に記録する。

### 5.6 Repair

`repair` は、`check` が返した失敗に対して、最小修正を適用する。
`repair` は対話側の状態に対して add-only に限る。局所的な欠損補完のみを扱い、たとえば不足参照の補完や不足 Condition の追加が該当する。既存 `binding_set` の再束ねが既存要素の無効化または差し替えを伴う場合は `revise` とする。

### 5.7 Revise

`revise` は、既存の束縛や前提集合そのものを改訂する。
この操作は append-only を守り、旧状態を削除しない。改訂は supersede relation を Sidecar に明示する追記として表現する。supersede relation は Sidecar の記録から回復可能でなければならない（たとえば action_trace または Condition-linked reference として残す）。たとえば、誤って採用した前提の差し替え、別の比較単位への切り替え、Condition 束の再編などが該当する。

### 5.8 Re-entry

`repair` または `revise` の後は、必ず `select` / `check` に戻る。
本 Annex の出口は compare そのものではなく、compare に進める状態の回復である。
open loop に無期限に留まり続けてはならない。
同一 failure に対する回復が `blocking_missing` を縮減できない場合は、open loop を打ち切り、`blocking_missing` を返して停止する。

NOTE: 打ち切りの具体的な反復上限や timeout は、modules / playbooks 側で定義する。

### 5.9 Minimal examples

#### Example A: missing eval frame

1. `ask`: `eval_frame_ref` が未解決である
2. `assert`: `eval_frame_ref` を束縛する
3. closure check: 必要参照が揃ったことを確認する
4. `select`: view / basis / condition を確定する（`agreed_condition_ref` から relevant set を正規化する）
5. `check`: 成立し、compare へ進む

#### Example B: repairing Condition missing

1. `check`: `undefined_type=U3` かつ `diagnostic_reason=ConditionMissing` を返す
2. `ask`: 不足条件を `open_questions` に追加する
3. `assert`: Condition を追記する（add-only）
4. `select`: `agreed_condition_ref` から relevant Condition を束ね直し、`condition_ref` に正規化する
5. `check`: 成立し、compare へ進む

#### Example C: revising a misadopted premise

1. `check`: `undefined_type=U2` かつ `diagnostic_reason=BasisIdMismatch` を返す
2. `revise`: 誤採用の前提に supersede relation を付与し、正しい前提を追記する
3. closure check: 必要参照が揃ったことを確認する
4. `select`: 更新後の `agreed_condition_ref` から relevant set を正規化する
5. `check`: 成立し、compare へ進む

## 6 Mapping to v0.1

本 Annex の最終目的は、v0.1 の `select` / `check` に戻って比較を前に進めることである。

対話で集めた Condition は `agreed_condition_ref`（候補束）として束ね、`select` が relevant set を確定して v0.1 の `condition_ref` に正規化する。対話で束縛した参照は `binding_set` として残し、v0.1 の ArtifactRef / SidecarRef の解決に使う。不成立は、v0.1 の二段返し `U1–U4 + diagnostic_reason` で止める。止めた結果は Sidecar に残し、次の `ask` `repair` `revise` の入力にする。

対話側の失敗は、v0.1 の型エラーを置き換えない。対話側は、v0.1 の失敗を「何が未確定か」という観測として扱い、次の参照確定へ戻す。Missing は `ask` の入力になり、Mismatch は `binding_set` または `agreed_condition_ref` の見直しを要求し、必要に応じて `select` による `condition_ref` の再正規化を要求する。Violation は手続の取り直しを要求する。

### Conformance

実装が本 Annex に準拠すると言うための最小要件は次である。

1. `dialogue_state_ref` から `space_id` `basis_id` `eval_frame_ref` `binding_set` `open_questions` `agreed_condition_ref` `adopted_evidence_refs` を解決できること
2. `operation_kind` として `ask` `assert` `repair` `revise` を Sidecar に記録できること
3. `repair` は対話側の状態に対して add-only であること。既存要素の無効化を伴う操作は `revise` として記録し、supersede relation を Sidecar から回復可能に残すこと
4. `agreed_condition_ref` は `condition_ref` の候補束として扱い、`select` による正規化を経て v0.1 の `condition_ref` に接続すること
5. 失敗は v0.1 の二段返しを尊重し、第一成分 U1–U4 を改変しないこと
6. v0.1 の `commit` と対話操作を混線させないこと

### Versioning

本 Annex は v0.2 以降の追加であり、v0.1 本文を変更しない。
予約語の追加は後方互換で行う。意味変更または必須条件の変更はメジャーアップとする。

## 7 Deferred to Modules

以下の論点は本 Annex の規範範囲に含めず、v0.2 modules / playbooks に委譲する。

1. `dialogue_state_ref` の再構成に必要な operation ごとの最小記録フィールド（B）
2. open loop の反復上限（回数・timeout）（F）
3. `repair` と `revise` の機械判定の自動検出ロジック
4. closure check の充足判定の実装手順
5. challenge-and-replay との closed loop 側の接続手順
