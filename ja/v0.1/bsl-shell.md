# BSL Shell v0.1.39（実装非依存の操作語彙）

## Changelog

- v0.1.1: minor（EvalFrameRef要件明確化、NOTE整流、型エラーのグルーピング、mapping_kind制約の明示）
- v0.1.2: 型エラー一覧に典型原因・最小修正を追記（回復手順の明示）。OrderingBreak/ContextLeakの注釈を補強
- v0.1.3: recoverableフラグ、anti-pattern（8本）、参照優先順位（衝突解決ルール）を追加
- v0.1.4: ActionTrace表記を`Reading(kind="action_trace")`に統一。selectにEvalFrameRef requireを追加。SpaceIdMismatch recoverableに補足
- v0.1.5: 標準パイプに required refs を追加。delta inspection に NOTE（判定分離）を再掲。select ensure に Φ を明示
- v0.1.6: select ensure に Φ 拘束の帰結を追記。maintenance に order/commit 役割区別 NOTE を追加
- v0.1.7: open.ensure の温度統一。select の LEM-4 を NOTE に移動。order/maintenance NOTE で「再現性の固定」語句を統一
- v0.1.8: compare produces に`kind="delta"`を追加。maintenance compare を`baseline_view_ref`参照に変更
- v0.1.9: select NOTE を診断優先順位への参照に変更（運用導線の統一）
- v0.1.10: check produces に`kind="check_result"`を追加。参照優先順位の view に「破棄可能」を追記
- v0.1.11: perturb ensure を Condition に統一（Variant は NOTE で将来候補として言及）。commit produces に`kind="diff_ref"`を追加
- v0.1.12: EvidenceAtom NOTE のフィールド名を`Reading.kind`に統一。open.ensure を「導入ログとして」に修正
- v0.1.13: capture produces に`kind="observation"`を追加。未指定時の正規化ルール NOTE を追加
- v0.1.14: capture NOTE に Condition の多義性明示（observation/perturb 両用）を追加
- v0.1.15: capture NOTE に condition_ref への束ね方（select で確定）を追加
- v0.1.16: select ensure に condition_ref の定義（relevant Condition を束ねた参照）を追加
- v0.1.17: ConditionMissing / ContextLeak 最小修正に condition_ref 確定手順を追加
- v0.1.18: v0.1/v0.2境界を明文化（modules分離、予約語の追加、肥大化ポイントの退避）
- v0.1.19: Contract/EffectDeclarationRef/undefined_type写像を追加（BSL_9 v0.7との接続）
- v0.1.20: PhiMismatch を undefined_type 分類から外し diagnostic_reason として扱う形式に変更。capture produces の区切りを ＋ に統一
- v0.1.21: 失敗の2段返し（undefined_type＋diagnostic_reason）をv0.1固定として明文化。check ensure を整理
- v0.1.22: 双方向・発話（dialogue）をv0.1スコープ外として明示。v0.2 Annex（予約語・拡張点）への導線を追加
- v0.1.23: undefined_type への直接写像を規範外とし、停止点決定規則（U1→U2→U3→U4優先）を規範として固定。diagnostic_reason フォーマットを非規範 NOTE で追加
- v0.1.24: 写像表を「代表例（非網羅）」として明示。U3/U4割当てを検出ゲート基準で固定。4章↔5.3の相互参照を追加
- v0.1.25: EvidenceAtom NOTE に根拠参照束（approval_ref/authority_ref/diff_ref/basis_ref）と参照型制約（resolvable/as_of束縛）を追加。check ensure に pass/fail 対称記録を追加。commit require に approval/authority/diff/basis 要件を追加。commit produces に basis_ref を追加。型エラー一覧に DiffRefMissing/BasisRefMissing を追加。check に check_result 最小フィールドの NOTE を追加（BSL_9 v1.1 との接続）
- v0.1.26: ArtifactRef に artifact_role の意味論的役割を明確化する NOTE を追加（型宣言としての位置づけ、required refs および non-comparable conditions との接続）。role固有の必須参照欠落時の停止点確定を明示（停止点確定規則への参照）
- v0.1.27: diagnostic_reason 推奨語彙に「運用上の崩壊モード（増幅ループ）」を最小追加し、具体手順は v0.2 playbooks に委譲する導線を追加
- v0.1.28: undefined_type の単一値原則を強化（U4併記削除、diagnostic_reason へ委譲）。停止点確定と診断順序の役割分離を明確化。典型例の整理（U1をcross_spaceに寄せ、欠落をU3/U4に整理）。用語の厳密化（ゲート→検出段、5.3の責務明確化）
- v0.1.29: role固有必須参照の停止点例を写像表と整合（basis_id→U2、trace_id→U4）
- v0.1.30: 5.3の責務をdiagnostic_reason確定に明確化し、undefined_type確定は4章参照に統一（因果の誤読防止）
- v0.1.31: undefined_type説明の曖昧表現を除去（検出段→停止点確定規則による評価順に統一）
- v0.1.32: 写像表NOTEの旧い言い方を除去（検出段基準→停止点確定規則に完全統一）
- v0.1.33: 表記の一貫性向上（数式記号をLaTeX記法に統一、check節NOTEを4章参照に簡略化）
- v0.1.34: 細部の一貫性向上（写像節の重複を2段返し参照に簡略化、OrderingMissing表記統一、U2典型例にbasis_id追加）
- v0.1.35: EffectDeclarationMissing定義の拡充（resolvable違反明示、典型原因網羅、recoverableの意味論的明確化）
- v0.1.36: 意味論の接続強化（EffectDeclarationRefに運用起因明示、compareにフォーマット差＝Δ明文化）
- v0.1.37: recoverableの意味論的定義を型エラー一覧冒頭に追加（「必ず回復できる」誤読を防止）
- v0.1.38: Gateとcheckの規律をスコープ冒頭に要約（Ruleは規範、NOTEは補助説明）
- v0.1.39: スコープ冒頭にGate/Evidence公理の要約を追加。秘匿禁止Rule→境界NOTE（情報分類は外部仕様レイヤの責務）に置換。gate Ruleにcheck_result種別を明示

---

## 0. スコープ（v0.1）

目的は「実装（CLI/API/ライブラリ）ではなく、操作の意味と契約（require/ensure）だけ」を定義すること。

比較・評価は評価フレーム$Φ=(\mathcal{I},\mathcal{A},\mathcal{C},\mathcal{O})$が閉じていることを前提にし、セッション内で暗黙変更しない。
（Ref: Core Appendix A.4.5, AX-8）

SSOTはStructureを汚染せず、読みはViewとSidecar（Basis/Condition/Ordering）で成立させる。
Rule: No gate without a check report (`Reading(kind="check_result")`).
Rule: No check without declared basis and constraints（Condition内の判定対象）.
NOTE: Gateは運用儀礼ではなく、統合の安全を担保する制御点である。
Rule: Evidence is append-only.
NOTE: Evidenceは再参照・再配布され得る。情報分類とアクセス制御は外部仕様レイヤの責務であり、Evidenceの内容はその制約下で設計される。

v0.1は「語彙＋契約（require/ensure）＋失敗型（診断優先順位）」を規範として固定する。
rules/sandboxes/playbooks等の機械運用資産はv0.2 modulesとして別仕様に分離する。
また、alignmentやfingerprint等の肥大化しやすい論点は、v0.1では"予約語（参照名）"の導入に留め、意味や手順はv0.2以降で定義する。

- NOTE: 発話・対話（dialogue）や双方向プロトコルは v0.1 の対象外とする。v0.2 では Sidecar/Contract の拡張（Annex）として追加し得るが、v0.1本文（語彙＋契約＋失敗型）の意味は変更しない。

---

## 1. 基本型（概念型）

- ArtifactRef
  - 必須: `artifact_id`, `artifact_role`(`ssot|view|sidecar|evidence`), `space_id`
  - 条件付き必須: `basis_id`（roleが`view|sidecar`）, `trace_id`（roleが`evidence`）
  - NOTE: `artifact_role`はShellにおける「型宣言」である。roleは、各operationが要求する参照束（required refs）と、比較禁止条件（non-comparable conditions）を決定する。
  - NOTE: role固有の必須参照（例: view/sidecarの`basis_id`, evidenceの`trace_id`）が欠ける場合、`check`はMissingを返し、停止点は停止点確定規則（U1→U2→U3→U4）に従って確定する（例: `basis_id`欠落は典型的にU2、`trace_id`欠落は典型的にU4）。

- EvalFrameRef（Φ参照）
  - 既定: $Φ=(\mathcal{I},\mathcal{A},\mathcal{C},\mathcal{O})$を分解参照する
    - `identity_criteria_ref`（$\mathcal{I}$）, `evidence_adoption_ref`（$\mathcal{A}$）, `condition_ref`（$\mathcal{C}$）, `ordering_ref`（$\mathcal{O}$）
  - 代替（任意）: `eval_frame_ref`（単一参照キー）
    - 実装は eval_frame_ref から $(\mathcal{I},\mathcal{A},\mathcal{C},\mathcal{O})$ を復元可能でなければならない
  - ルール: 分解参照と eval_frame_ref の併記はしない（どちらか一方）

- EffectDeclarationRef（外部依存の宣言参照）
  - 必須: `effect_id`
  - 任意: `kind`（toolcall/external_lookup/model_version等）, `ref`（参照対象）, `as_of`（参照時刻・版）
  - ルール: operation が外部依存を持つ場合、EffectDeclarationRef が解決できなければならない（resolvable）
  - NOTE: 外部要因（SaaS停止等）で一時的に参照不能な場合も、v0.1では同じ失敗型に含める。事情は`diagnostic_reason`とdetailsで表現し、手順の分岐はv0.2 playbooksへ委譲する。
  - NOTE: OS層は「参照できること」だけを要求し、payload 形式は Implementation 層で決定してよい（Ref: BSL_9 6.3）
  - NOTE: Shell の EffectDeclarationRef は、Space Metadata の `effect_declaration` に対応する参照表現である。

- EvidenceAtom（Evidence軸の要素）
  - `Reading`, `Condition`, `Ordering`（append-only）
  - NOTE: ActionTrace は独立Atomではない。`Reading.kind="action_trace"`として表現する。
  - NOTE: compare が produces する `Reading(kind="delta")` も EvidenceAtom に含まれる Reading である。
  - NOTE: 採用（adoption）に関わる根拠参照束は、少なくとも `approval_ref` / `authority_ref` / `diff_ref` / `basis_ref` を含む。
  - NOTE: `diff_ref` / `basis_ref` は解決可能な参照（resolvable reference）であり、参照先を一意に解決でき、as_of 束縛（版・時刻・コミット等）を付与可能でなければならない。

---

## 2. 基本コマンド（9個）と分類

### read

1. `open`（参照の導入）
   - require: ArtifactRefが最小メタデータを満たす（role/space、条件付きでbasis/trace）
   - ensure: 導入ログとして`Reading(kind="action_trace")`相当のEvidenceを追加する（成功・反映を含意しない）
   - produces evidence: `Evidence.Reading(kind="action_trace")`（導入ログ）

2. `select`（View選択と読みの成立条件の確定）
   - require: `ssot`と`sidecar`が同一`space_id`で、View適用に必要な`basis_id`が確定している
   - require: EvalFrameRef（分解参照またはeval_frame_ref）が解決できる
   - require: operation が外部依存を持つ場合、EffectDeclarationRef が解決できる
   - ensure: 「どのViewを、どのSidecar条件で、どのΦで読むか」を確定し、比較の前提を明示化する（以後、compareはこのΦに拘束される）
   - ensure: condition_ref は、captureで集めた Condition のうち relevant な集合を束ねた参照である
   - produces evidence: `Evidence.Reading(kind="action_trace")`（選択したview/basis/condition参照。成立の宣言であり、結果の正しさを含意しない）
   - NOTE: select の成果は Contract（OperationScope + Φ + RequiredEvidence + EffectDeclaration）の確定である（Ref: Core DEF-Contract）
   - NOTE: 比較へ入る前の一般条件（space/basisの一致条件）は「失敗優先順位 2（Space/Basis）」に従って診断する

### derive

3. `derive`（SSOT→Viewの再計算）
   - require: 入力が`ssot`と`sidecar`（basis/condition/ordering）で、依存方向を破らない
   - ensure: 同一入力なら同一出力（決定性）。出力は`view`として`basis_id`を持つ
   - produces evidence: `Evidence.Reading(kind="action_trace")`（deriveの入力参照と出力参照）
   - NOTE: derive は ssot 更新の根拠として許可されない（Ref: BSL_9 7.2）。更新は commit 経由で行う。

4. `perturb`（摂動：入力条件か写像を揺らす）
   - require: 摂動対象が`view`または`sidecar`に閉じ、SSOT（Structure）を汚染しない
   - ensure: 摂動条件を Condition として宣言し、比較に使う条件差を明示する
   - produces evidence: `Evidence.Condition`（摂動条件の追加）＋`Evidence.Reading(kind="action_trace")`
   - NOTE: 「Variant」は将来の用語候補（v0.1では型として定義しない）。摂動は Condition に落とす。

### compare

5. `compare`（Δ計算：同列比較）
   - require: 比較対象が同一`space_id`で、Φが一致し、`basis_id`が一致（または許容ルールが明示）している
   - ensure: 出力Δは「判定」ではなく、差分の事実（中立）としてEvidenceに残せる
   - produces evidence: `Evidence.Reading(kind="delta")`＋`Evidence.Reading(kind="action_trace")`
   - NOTE: v0.1は正規化しない。表現差（フォーマット差・表記ゆれ等）はΔとして検出し、許容誤差や正規化はalignment/fingerprint（v0.2）で扱う。
   - NOTE: 予約語（v0.1）: `alignment_ref`（整列の宣言参照）, `mapping_id`（整列に用いた写像の識別子）。意味・必須条件・許容ルールはv0.2で定義する。

6. `check`（型・契約・禁止則の検査）
   - require: 対象ArtifactRefが`open`済みで、必要な参照（space/basis/trace/ordering）が揃っている
   - ensure: 失敗は常に2段で返す：`undefined_type`（U1–U4, BSL準拠）＋`diagnostic_reason`（Shell拡張, informative）
   - ensure: pass/fail は同一形式の `Evidence.Reading(kind="check_result")` として Evidence Chain に追記されなければならない
   - ensure: `check_result` を追記できない場合、操作は不成立として U4 を返す
   - produces evidence: `Evidence.Reading(kind="check_result")`＋`Evidence.Reading(kind="action_trace")`
   - NOTE: 「止める」は観測設計である（4章「失敗の返し方（2段返し）」参照）。
   - NOTE: `check_result` は fail 時に `undefined_type` ＋ `diagnostic_reason` を最低限含まなければならない（集計・再現の安定性のため）

### write

7. `capture`（Evidenceの追記）
   - require: `trace_id`が存在し、append-onlyを満たす
   - ensure: `Reading`/`Condition`は削除・上書きせず、無効化は追記で表す
   - produces evidence: `Evidence.Reading(kind="observation")`＋`Evidence.Condition`＋`Evidence.Reading(kind="action_trace")`
   - NOTE: Reading.kind は拡張可。未指定のReadingは`kind="observation"`として正規化してよい。
   - NOTE: Condition は observation/perturb の両方に使う（区別は value 側で表現する）。v0.1では型を増やさない。
   - NOTE: 同一Φで比較するため、compare前に relevant Condition を condition_ref に束ねる（selectで確定する）。

8. `order`（Orderingの追記：観測順序の固定）
   - require: `Ordering`が単調増加でチェイン整合する（切断・巻き戻し禁止）
   - ensure: 以後の比較は、この`ordering_ref`を含むΦでのみ再現可能になる（再現性の固定）
   - produces evidence: `Evidence.Ordering`

9. `commit`（SSOT更新の明示手続）
   - require: Orderingで許可されたフェーズに閉じ、更新が差分参照（diff_ref）として残る
   - require: 外部作用を伴うSSOT更新は、`approval_ref` と `authority_ref` を欠いたまま成立させてはならない
   - require: 採用判断を伴うcommitは、`diff_ref` と `basis_ref` を伴って成立しなければならない
   - require: `mapping_kind` ∈ {`transfer`, `transform`, `manual`}
   - require: `mapping_kind` ∉ {`derive`, `observe`, `compose`}
   - ensure: ViewがSSOTを直接更新しない（逆流禁止）
   - produces evidence: `Evidence.Reading(kind="diff_ref")`＋`Evidence.Reading(kind="basis_ref")`＋`Evidence.Reading(kind="action_trace")`

---

## 3. 標準パイプ（3本）

### 1. maintenance（保守・再現性回復）

- required refs (minimum): `space_id`, `basis_id`, Φ(`EvalFrameRef`), `trace_id`, `ordering_ref`

```
open（ssot/sidecar/evidence）
  ↓
check（Φと必須キー）
  ↓
derive（view再計算）
  ↓
compare（baseline_view_refとのΔ）
  ↓
必要なら order → commit（手続きでSSOT更新）
```

根拠: SSOTと読みの分離、Orderingによる再現、writeの閉じ込め。

- NOTE: `order`は再現性の固定（観測順の拘束）であり、orderのordering_refを含むΦでのみ再現可能になる。`commit`は更新手続（diff_ref必須）である

### 2. delta inspection（差分検図：differential/metamorphic寄り）

- required refs (minimum): `space_id`, `basis_id`, Φ(`EvalFrameRef`), `ordering_ref`（deltaの再現が必要な場合）

```
open（対象A/B）
  ↓
select（同一View/同一条件の宣言）
  ↓
derive（A_view/B_view）
  ↓
compare（Δ抽出）
  ↓
capture（ΔをReadingとして残す。判定は別レイヤ）
```

- NOTE: 出力Δは判定ではない（accept/rejectはPolicy/Behavior側で行う）

### 3. evidence capture（証跡収集：oracle弱い状況の足場作り）

- required refs (minimum): `trace_id`, `ordering_ref`（または order により生成）, `space_id`（比較を想定する場合）, Φ（比較を想定する場合）

```
open（対象とtrace）
  ↓
order（観測開始）
  ↓
capture（Reading/Conditionを逐次追記）
  ↓
derive（観測に基づくviewスナップショット）
  ↓
compare（必要なら基準viewと）
```

根拠: Evidence Chain append-onlyとOrderingのSSOT性。

---

## 4. 型エラー一覧（v0.1）

NOTE: 各型エラーの `recoverable` は「同一$Φ$で再実行できる参照束を再確定できる可能性」を指し、外部要因による一時不成立を排除しない。運用起因の参照不能は `diagnostic_reason` とdetailsで表現する。

### 評価フレーム

- `EvalFrameRefMissing`
  - compare/check に必要な Φ参照（分解参照または eval_frame_ref）が存在しない
  - recoverable: yes
  - 典型原因: compare/check を先行し、eval_frame_ref（または ℐ/𝒜/𝒞/𝒪 参照）を導入していない
  - 最小修正: eval_frame_ref（または4参照）を sidecar/セッションに付与し、select→compare の順にする
- `PhiMismatch`
  - Φの要素（ℐ,𝒜,𝒞,𝒪）参照が一致しない
  - recoverable: yes（同一Φに揃えられる場合）
  - 典型原因: 左右で identity_criteria_ref/condition_ref/ordering_ref などが異なるまま比較している
  - 最小修正: 同一 Φ に揃えて再実行し、揃えられない差は space/basis を分けて扱う
- `EffectDeclarationMissing`
  - operation が外部依存を持つが、`EffectDeclarationRef`が存在しない、または解決不能（resolvable違反）
  - recoverable: yes
  - 典型原因: derive/compare が外部ツール・辞書・モデル等に依存するが、`effect_declaration`が未宣言、または`effect_id/ref/as_of`が確定していない
  - 最小修正: `effect_declaration`を宣言し、`effect_id/ref/as_of`を確定させて再実行する
  - NOTE: v0.1では「欠落」と「外部参照失敗（解決不能）」を同一型で扱う。v0.2以降で`EffectDeclarationUnresolvable`（ref参照不能）への分割を検討してよい
  - NOTE: recoverableは「参照束を再確定して同一$Φ$で再実行できる可能性」を指す。外部SaaSの一時停止など運用起因の参照不能は`diagnostic_reason`（例:`effect_unresolvable`）とdetailsで表現する

### Space/Basis

- `SpaceIdMismatch`
  - `space_id`不一致（比較単位が違う）
  - recoverable: no（同一spaceでの比較としては回復不能）
  - 典型原因: 空間（成立条件の前提集合）が異なる対象を直接 compare している
  - 最小修正: 同一 space に揃える（移送/変換で新 space を作る等）か、比較自体を中止する
- `BasisIdMissing` / `BasisIdMismatch`
  - view/sidecarで`basis_id`欠落、または異なるbasisを暗黙に混ぜた比較
  - recoverable: yes
  - 典型原因: view/sidecar に basis_id が付いていない、または別 basis 由来の view を混在させている
  - 最小修正: basis_id を明示し、同一 basis で derive し直してから compare する

### Evidence 連鎖

- `TraceIdMissing`
  - evidenceに`trace_id`がない（証跡連鎖に入っていない）
  - recoverable: yes
  - 典型原因: capture 対象の evidence が trace_id を持たない（参照できない証跡になっている）
  - 最小修正: trace_id を割り当てた evidence を用意し、open→order/capture をやり直す
- `OrderingMissing` / `OrderingBreak`
  - `ordering_ref`欠落、sequence_number不連続、previous参照切断
  - recoverable: yes
  - 典型原因: order を開始せずに capture した、または previous/sequence が途切れる追記を行った
  - 最小修正: order で起点を作り、単調増加＋previous 連結を満たす形で追記する。
    破断が回復不能な場合は新チェインを開始し、旧チェインとの関係を Condition として記録する。
- `NonAppendOnly`
  - Reading/Condition/Orderingの上書き・削除が検出される
  - recoverable: no
  - 典型原因: 既存エントリの編集・削除、ID再利用、過去の ordering を書き換えて整合させようとした
  - 最小修正: 追記で訂正（無効化/置換の宣言）し、元エントリは不変のまま残す

### 役割・条件

- `ArtifactRoleMismatch`
  - `ssot`と`view`を同列に比較する等、役割に対する操作の取り違え
  - recoverable: yes
  - 典型原因: ssot を view として扱う、view を ssot として commit する等、role 前提を崩している
  - 最小修正: ssot は derive の入力、view は compare の対象、更新は commit 経由に固定する
- `ConditionMissing` / `ContextLeak`
  - Δ計算に必要なConditionが明示されない、またはCondition/Context境界が崩れる
  - recoverable: yes
  - 典型原因: 摂動条件や環境依存（暗黙の前提）が sidecar/condition_ref に落ちていない、
    または Context（外側レイヤ）が Condition（Evidence 軸内）に混入している
  - 最小修正: 条件を Condition として capture（または perturb）し、relevant Condition を condition_ref として select で確定したうえで、同条件で derive→compare を再実行する

### 依存・手続

- `DependencyViolation`
  - 禁止依存（例: Flow→Evidenceの混入、Flow→Behaviorの内包）
  - recoverable: yes
  - 典型原因: 中立な Evidence に判定/意図を混入、または禁止された方向で参照・更新が起きている
  - 最小修正: 判定は Policy/Behavior 側に分離し、Shell では参照・追記・手続のみに戻す
- `MappingKindViolation`
  - commit の mapping_kind が許容集合（transfer/transform/manual）に入っていない
  - recoverable: yes
  - 典型原因: mapping_kind に derive/observe/compose を指定、または mapping_kind 自体が未確定
  - 最小修正: transfer/transform/manual のいずれかを明示し、diff_ref を伴って commit する
- `DirectWriteFromView`
  - viewがSSOTを直接更新（commit手続き不在）
  - recoverable: yes
  - 典型原因: view 側の編集結果を ssot に反映した扱いにしている（手続と証跡が無い）
  - 最小修正: 変更は diff として切り出し、commit で ssot 更新し、更新後に derive で view を再生成する
- `DiffRefMissing` / `BasisRefMissing`
  - 採用判断を伴う commit の根拠参照束（diff_ref / basis_ref）が欠落
  - recoverable: yes
  - 典型原因: commit が diff_ref / basis_ref を欠いたまま実行されている
  - 最小修正: diff_ref（比較対象への参照）と basis_ref（判断基準への参照）を明示して commit する
  - NOTE: diagnostic_reason としては `diff_ref_missing` / `basis_ref_missing` を使用（4章「失敗の返し方」参照）。型エラー名は PascalCase、diagnostic_reason は snake_case を用いる。

### 決定性

- `NonDeterministicDerive`
  - 同一入力（ssot+sidecar）からderive結果が変動（比較の前提が壊れる）
  - recoverable: yes
  - 典型原因: derive が時刻/乱数/外部状態/未固定依存に影響され、同一入力で出力が揺れる
  - 最小修正: 依存を sidecar に固定化（または EffectDeclarationRef で宣言）し、derive を決定的にしてから compare を行う

### undefined_type への写像（BSL_9 7.6 との接続）

Shell の失敗は BSL_9 Checks の undefined_type（U1〜U4）に接続できる。
ただしv0.1では、型エラー名（ConditionMissing、ContextLeak、ArtifactRoleMismatch等）を undefined_type に「直接写像する」ことは規範にしない。
型エラー名は diagnostic_reason（informative）として返し、undefined_type は停止点（ゲート）として固定する。

以下は、停止点U1–U4が典型的に選ばれる代表例である（非網羅）。型エラー→undefined_typeの直接写像は規範にしない。

| 代表例（Shell型エラー） | undefined_type | 代表理由 |
|---------------|----------------|------|
| SpaceIdMismatch | U1（cross_space） | space_id 不一致で直接比較禁止 |
| BasisIdMissing / BasisIdMismatch | U2（basis_mismatch） | basis_id 不一致で座標系が揃わない |
| EvalFrameRefMissing / EffectDeclarationMissing | U3（frame_not_closed） | Φ または外部依存が未閉包 |
| TraceIdMissing / OrderingMissing / OrderingBreak / NonAppendOnly | U4（evidence_missing） | evidence 連鎖不成立で再現不能 |

- NOTE: undefined_type の停止点確定規則と diagnostic_reason の確定については、本節末尾の「失敗の返し方（2段返し）」を参照。
- NOTE: U3/U4の割当ては意味論ではなく、停止点確定規則（U1→U2→U3→U4）の評価順で「最初に不成立となった側」を返す（diagnostic_reasonの確定は5.3）。
- NOTE: 以下は、各 undefined_type が典型的に対応しうる失敗モードの例である（非網羅）。実際の undefined_type は停止点確定規則で確定し、意味論的分類ではない。

| undefined_type | 典型的な失敗モード例（diagnostic_reason 側で詳細化） |
|---|---|
| U1（cross_space） | space_id_mismatch（cross-space compare の試行） |
| U2（basis_mismatch） | basis_id mismatch / scope/role mismatch（許可されない operation_kind） |
| U3（frame_not_closed） | frame/boundary missing-or-violation（eval_frame/effect/approval 等の未閉包、境界違反） |
| U4（evidence_missing） | evidence chain broken（trace_id 追跡不能、ordering/append-only 破綻、failログ追記不能） |

- NOTE: U1/U2 が回復不能な場合は Semantic Discontinuity（Core A.4.4）に分類される
- NOTE: U3/U4 は原則として回復可能（欠落要素を補完可能）
- NOTE: effect_declaration の欠落/参照不能は原則として U3（frame_not_closed）に分類する。
  参照不能（ref不成立）の事情は `diagnostic_reason`（例: `effect_unresolvable` など）と details で表現する（undefined_type は単一値を維持）。
- NOTE: `PhiMismatch`（閉じているが一致しない）は undefined_type（U1–U4）の分類そのものではない。
  Shell では undefined_type を U1–U4（BSL準拠の大分類）で固定し、`diagnostic_reason`（informative）として `PhiMismatch` / `ContractMismatch` を付与する。

### 失敗の返し方（2段返し：v0.1固定）

Shell は「どこで止めるか」を固定するため、失敗を常に次の2段で返す。

- `undefined_type`: U1–U4（BSL準拠の大分類。比較の不成立を表す）
- `diagnostic_reason`: Missing / Mismatch / Violation 等（Shell拡張。回復手順の入口）

`undefined_type` は停止点確定規則に従い、U1→U2→U3→U4の順で評価して最初に不成立となった停止点を返す。

NOTE: `undefined_type` は「比較が前に進まない理由（停止点）」を表す。`diagnostic_reason` は「最小修正の方向（何を確定し直すか）」を表す。停止は守りではなく観測設計である。
NOTE: `diagnostic_reason` は v0.1 では文字列でもよい。将来の集計を想定する場合、kind(Missing|Mismatch|Violation|Other) / code(型エラー名) / details(参照ID等) の2〜3層に分けてもよい（非規範）。
NOTE: 停止点は `undefined_type` として U1→U2→U3→U4 の優先で確定する。原因は `diagnostic_reason` として 5.3「失敗優先順位」（Missing優先など）で確定する。両者は役割が異なる。
NOTE: 推奨語彙（機械判定可能な失敗モード）: `call_without_effect` / `approval_missing` / `basis_not_closed` / `ssot_pollution_attempt` / `mapping_mismatch` / `amplification_loop` / `noisy_neighbor` / `backfill_overrun`。
NOTE: 推奨語彙（欠落）: `diff_ref_missing` / `basis_ref_missing`（採用判断を伴うcommitの根拠参照束が欠落）
NOTE: `amplification_loop` は「遅延→timeout→retry により負荷が自己増幅している」崩壊モードを指す。v0.1では語彙のみ規範化し、遮断・レート制限・優先度分離などの回復手順は v0.2 modules/playbooks に委譲する。
NOTE: `noisy_neighbor` は「一部ワークロードの高負荷が他を巻き込み、比較/採用の経路を不安定化させる」崩壊モードを指す（分離設計は v0.2）。
NOTE: `backfill_overrun` は「低優先の一括処理（バックフィル等）が資源予算を食い潰し、通常系の成立条件を壊す」崩壊モードを指す（レート制限は v0.2）。

---

## 5. Guardrails（anti-pattern／参照優先順位）

### 5.1 Anti-pattern（8本）

1. `open→compare`（select/Φ導入を飛ばす）
   - 失敗: `EvalFrameRefMissing`（外部依存がある場合は `EffectDeclarationMissing` も起きる）
   - 最小修正: eval_frame_ref（または ℐ/𝒜/𝒞/𝒪）を導入し、外部依存がある場合は effect_declaration も確定させて、`select→compare` の順に戻す

2. space跨ぎの直接比較（「似ているから」でcompare）
   - 失敗: `SpaceIdMismatch`
   - 最小修正: 同一spaceへ揃える（transfer/transformで新spaceを作る）か、比較自体を中止する

3. basis混在の比較（別basis由来viewを同列にcompare）
   - 失敗: `BasisIdMismatch` / `BasisIdMissing`
   - 最小修正: basis_id を確定し、同一basisで `derive` し直してから `compare`

4. `order`なしの連続`capture`（観測をつなげない）
   - 失敗: `OrderingMissing` / `OrderingBreak`
   - 最小修正: `order`で起点を作り、単調増加＋previous連結で追記（回復不能なら新チェイン＋関係をCondition記録）

5. Evidenceの"後編集"で辻褄合わせ（上書き・削除・差し替え）
   - 失敗: `NonAppendOnly`
   - 最小修正: 訂正は追記（無効化/置換宣言）で行い、原エントリは不変のまま残す

6. derive結果を根拠にssotを書き換える（commitを省略）
   - 失敗: `DirectWriteFromView`
   - 最小修正: 変更は diff として切り出し、`commit`経由でssot更新し、その後`derive`でview再生成

7. mapping_kindに`derive/observe/compose`を指定してcommitする
   - 失敗: `MappingKindViolation`
   - 最小修正: mapping_kind を `transfer/transform/manual` に明示し、diff_ref を伴って `commit`

8. 判定・意図をEvidenceへ混入（ReadingにOK/NGや因果を直接書く）
   - 失敗: `DependencyViolation`
   - 最小修正: Evidenceは中立のまま保持し、判定はPolicy/Behavior側へ分離する

### 5.2 参照の優先順位（Reference Resolution）

衝突時は上位を採用し、下位は「派生」「観測」「提示」として扱う。

1. Φ（EvalFrameRef）
2. `space_id`
3. `basis_id`
4. `effect_declaration`（外部依存の宣言）
5. `ordering_ref`
6. `trace_id`
7. `ssot`（Structureの事実）
8. `sidecar`（Condition/Ordering/派生情報）
9. `view`（常に再計算物。破棄可能）
10. `Reading`（観測・Δ。判定ではない）

NOTE: effect_declaration は ordering_ref より上位。外部依存が固定されていないと、ordering_ref があっても再現性が保証されないため。

### 5.3 失敗優先順位（診断の先頭条件）

診断は次の順で短絡する（成立条件→操作妥当性→再現性）。
同一段内では Missing を先に返し、Mismatch は Missing が無い場合に評価する。

1. `EvalFrameRefMissing` / `EffectDeclarationMissing`（まず閉包の欠落を返す）、欠落が無い場合に `PhiMismatch` を評価する
2. `SpaceIdMismatch` / `BasisIdMissing`、次いで `BasisIdMismatch`
3. `TraceIdMissing` / `OrderingMissing` / `OrderingBreak` / `NonAppendOnly`
4. `ArtifactRoleMismatch` / `ConditionMissing|ContextLeak`
5. `DependencyViolation` / `MappingKindViolation` / `DirectWriteFromView`
6. `NonDeterministicDerive`

NOTE: 5.3の順で検査し、最初に不成立となった事項を `diagnostic_reason`（Missing/Mismatch/Violation）として返す。停止点（`undefined_type`）の確定は 4章の停止点確定規則（U1→U2→U3→U4優先）に従う。
NOTE: 停止点の確定規則（U1→U2→U3→U4優先）と、診断の順序（5.3）は役割が異なる。停止点確定は 4章「undefined_type への写像」および「2段返し」を参照。

---

## 6. v0.1での割り切り（不採用の明示）

ここで定義したのは「語彙＋契約＋失敗型」であり、最適化・網羅・自動修復は扱わない。

ただし、metamorphic/differential/property-based/model-basedの"入口"として必要な最小部品（摂動、同列比較、契約、oracle弱の失敗型）は、このShell語彙で揃う。

NOTE: 「oracle弱」は、テスト論における Test Oracle Problem の文脈で、正解判定（oracle）が強く定義できない／コスト的に置けない状況を指す。v0.1では用語としてのみ使用し、判定ロジックは定義しない（→ Sandboxes/Glossary候補）。

### 6.1 v0.1/v0.2 境界（固定）

- v0.1（規範）: require/ensure、参照優先順位、失敗優先順位、型エラー（recoverable＋最小修正）、禁止経路（直接比較/逆流/非append-only）を固定する。
- v0.2（modules）: 規範を機械運用に落とす資産（rules/sandboxes/playbooks）を別仕様として追加する。v0.1本文は"読めば一致する"を優先し、"回せば一致する"はmodules側で扱う。
- NOTE: dialogue（発話・対話）は v0.2 以降で Annex として追加し得る（Sidecar/Contract/operation_kind の拡張）。v0.1の operation_kind は read/derive/compare/write に限定する（2章の分類に準拠）。

### 6.2 v0.2へ送る論点（予約語のみ許容）

- alignment: `alignment_ref`, `mapping_id`の意味、必須条件、許容ルール、許容誤差、正規化はv0.2で定義する（v0.1は予約語のみ）。
- fingerprint/正規化: ハッシュ、正規化手順、安定化ポリシーはv0.2で定義する（v0.1は`Reading.kind`の正規化まで）。
- provenance写像: PROV等への写像、Agent分類、監査・セキュリティ連携はv0.2以降で定義する（v0.1はEvidenceAtom＋kindを正準とする）。

### 6.3 v0.2 Annex（dialogue拡張：予約語のみ）

- Sidecar: `dialogue_state_ref`（対話進行に伴う参照束縛・未解決問い・合意済み前提・採用Evidence の追記履歴）
- operation_kind拡張（v0.2以降）: `ask`, `assert`, `repair`, `revise`（v0.1の外）
- Contract拡張（v0.2以降）: `reference_binding_policy_ref`, `repair_priority_ref`, `incomparable_reason_ref`
- NOTE: 上記は「予約語」だけを置く。意味・必須条件・ログ粒度・失敗型への写像は v0.2 Annex 側で定義する。
- NOTE: 詳細仕様は BSL Shell Dialogue Annex v0.2 を参照。
