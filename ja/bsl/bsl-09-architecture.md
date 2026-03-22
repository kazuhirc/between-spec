# BSL_9. Architecture 仕様

**Version: v1.1.3**

---

## Core Dependency

本章が依拠するCoreの定義を以下に示す。

| Core節 | 本章での役割 |
|--------|-------------|
| A.1 / A.2（三軸×三層） | OS層（BSL）の構造基盤 |
| A.3（依存ポリシー） | レイヤ間の参照方向 |
| A.5（View / Sidecar） | 実装層への最小インターフェース |
| A.6（外側レイヤ） | 判断構造の位置づけ |

BSL独自の定義：Layer / Boundary / Slot

---

## 1. Purpose（この章の目的）

本章は、BSLが担う「意味のOS層」が、実装層やアプリケーション層とどのように関係し、どこまでが固定され、どこからが自由かを形式的に定義する「境界仕様」である。

BSLが扱うもの（意味構造）と、扱わないもの（表現・実装）を明確に分離する。

仕様化の範囲：
- レイヤ構造（OS / Implementation / Application）
- 境界条件（固定／自由）
- Slot（抽象マッピング構造）
- 依存ポリシー

仕様化の範囲外：
- 実装層の詳細仕様 → BSL Annex / Sandboxes
- UI／操作／アルゴリズム → Sandboxes
- OSS公開／特許戦略 → 別文書

---

## 2. Layer Structure（レイヤ構造）

### 2.1 三層モデル

```
┌─────────────────────────────────────────┐
│      Application Layer                  │
│      アプリケーション層                  │
│      問題解決・UX提供                    │
└───────────────┬─────────────────────────┘
                │ 利用
                ▼
┌─────────────────────────────────────────┐
│      Implementation Layer               │
│      実装層                              │
│      表現 / 操作 / データ形式            │
└───────────────┬─────────────────────────┘
                │ 準拠
                ▼
┌─────────────────────────────────────────┐
│      OS Layer (BSL)                     │
│      意味OS層                            │
│      Flow / Behavior / Evidence /       │
│      Variant / Design History /         │
│      Operation / Continuity             │
└─────────────────────────────────────────┘
```

### 2.2 各層の責務

| 層 | 名称 | 責務 | BSLとの関係 |
|----|------|------|-------------|
| OS層 | BSL | 意味の座標を定義 | 本仕様 |
| 実装層 | 任意 | 意味を現場形式に変換 | BSLに準拠 |
| アプリ層 | 任意 | 問題解決・UX提供 | 実装層を利用 |

---

## 3. Boundary Definition（境界定義）

### 3.1 BSL（OS層）が固定するもの

| カテゴリ | 固定内容 | 参照 |
|----------|----------|------|
| 三軸構造 | Flow / Behavior / Evidence | BSL_2〜4 |
| 三層構造 | Element / Structure / Basis | Core A.2 |
| 外側レイヤ | Variant / Design History | BSL_5, 6 |
| 実行構造 | Operation / Continuity | BSL_7, 8 |
| 依存方向 | Evidence → Behavior → Flow | Core A.3 |
| 非破壊記録 | Evidence Chain append-only | Core A.5 |
| 同一性 | Meaning Identity / Variation | Core A.4 |
| 評価フレーム | Φ = (ℐ, 𝒜, 𝒞, 𝒪) の固定 | Core AX-8 |

評価フレーム Φ は、比較・評価・最適化が成立するための最小契約である。
Φ を構成する Identity 判定基準 ℐ、Evidence 採用条件 𝒜、Condition 𝒞、Ordering 𝒪 のいずれかが欠落または暗黙変更された場合、比較結果は再現不能となり、同列比較の対象とならない。

### 3.2 実装層が自由に決めるもの

| カテゴリ | 自由度 | 例 |
|----------|--------|-----|
| データ形式 | 完全自由 | JSON / YAML / SQL / ファイル |
| 表現形式 | 完全自由 | ベクタ / ラスタ / テキスト |
| 実装言語 | 完全自由 | 任意の言語・ランタイム |
| UI / 操作 | 完全自由 | CLI / GUI / API |
| Sidecar物理形式 | 完全自由 | ファイル / DB / クラウド |
| View挙動 | 完全自由 | フィルタ / ズーム / ハイライト |
| アルゴリズム | 完全自由 | 配置決定 / 差分検出 / 自動生成 |

### 3.3 境界条件（Boundary Conditions）

| ID | 条件 | 説明 |
|----|------|------|
| BD-1 | 意味の固定 | BSLが定義する意味座標は不変 |
| BD-2 | 表現の自由 | 表現・形式は実装層が自由に決定 |
| BD-3 | 依存方向 | 実装層 → OS層への準拠のみ（逆方向禁止） |
| BD-4 | 非侵食 | 実装都合がOS層仕様を歪めてはならない |

---

## 4. Data Model（データモデル）

### 4.1 Layer（レイヤ定義）

レイヤを形式的に定義する構造。補足の JSON Schema 例は BSL Annex を参照。

#### Example

```json
{
  "id": "LY001",
  "name": "os",
  "description": "BSL - 意味OS層",
  "components": ["Flow", "Behavior", "Evidence", "Variant", "DesignHistory", "Operation", "Continuity"],
  "depends_on": []
}
```

### 4.2 Slot（意味概念 → 実装データの写像）

Slot は、OS層の概念を実装層のデータに対応づける抽象マッピング構造。

- BSLは Slot の構造を定義する
- 実装層は Slot の payload形式を自由に決められる

#### Schema

```json
{
  "$defs": {
    "Slot": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "一意識別子（ハッシュ推奨）"
        },
        "type": {
          "type": "string",
          "description": "BSL概念名（例：Flow.Part, Evidence.Reading）"
        },
        "payload": {
          "description": "実データ（形式は実装依存）"
        },
        "evidence_chain": {
          "type": "array",
          "items": { "type": "string" },
          "description": "裏付けEvidence（append-only）"
        },
        "metadata": {
          "type": "object",
          "description": "実装固有のメタデータ"
        }
      },
      "required": ["id", "type"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | 一意 | 識別子（ハッシュ推奨） |
| type | string | ○ | BSL概念名 | 参考: Slot Type Catalog（Annex） |
| payload | any | - | 実装依存 | 実データ |
| evidence_chain | array | - | append-only | 裏付けEvidence |
| metadata | object | - | - | 実装固有情報 |

Slot.type にはBSL概念名（Flow.Part, Behavior.Step, Evidence.Reading 等）を使用する。標準的な type 一覧の参考例は BSL Annex「Slot Type Catalog」を参照。

### 4.3 Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| SL-C1 | Slot.type はBSL概念に対応すること | 意味座標の固定 |
| SL-C2 | payload形式は実装層が決定 | BD-2 |
| SL-C3 | evidence_chain はappend-only | Core A.5 |
| SL-C4 | Evidence/Behavior/Flowの依存方向を崩さない | Core A.3 |
| LY-C1 | 下位レイヤは上位レイヤに依存しない | BD-3 |

---

## 5. Dependency Policy（依存ポリシー）

### 5.1 レイヤ間依存

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| アプリ層 | 実装層 | ○ | 利用 |
| アプリ層 | OS層 | ○ | 概念参照 |
| 実装層 | OS層 | ○ | 準拠 |
| OS層 | 実装層 | × | 禁止（逆流） |
| OS層 | アプリ層 | × | 禁止（逆流） |
| 実装層 | アプリ層 | × | 禁止（逆流） |

### 5.2 BSL内部の依存（再掲）

```
Continuity
   ↑
Operation
   ↑
外側レイヤ（Variant / DesignHistory / Context）
   ↑
三軸（Evidence → Behavior → Flow）
```

---

## 6. Space Metadata（Space メタデータ）

BSL は既存の型宣言（JSON Schema / OpenAPI / Protobuf / TypeScript 型など）へ付与できる共通メタデータを定義する。
表現方法（vendor extension、独自キー、コメント等）は媒体に委ねるが、キーと意味は統一する。

### 6.1 必須キー

| キー | 型 | 説明 |
|------|-----|------|
| space_id | string, required | Space を識別する ID。同一性（Identity）の比較単位 |
| artifact_role | enum, required | 当該アーティファクトの役割: ssot / view / sidecar / evidence |

### 6.2 条件付き必須キー

| キー | 型 | 条件 | 説明 |
|------|-----|------|------|
| basis_id | string | required if artifact_role is view or sidecar | 解釈前提を識別する ID。basis_id が異なる成果物は直接比較してはならない |
| trace_id | string | required if artifact_role is evidence | 証跡連鎖を識別する ID。trace_id を欠く evidence は採用してはならない |

### 6.3 任意キー

| キー | 型 | 説明 |
|------|-----|------|
| space_type | enum/string, optional | Space の性質を表す分類。列挙は運用で拡張してよい。比較許可は space_id を基準とする |
| mapping_id | string, optional | Mapping を識別する ID。Space 間接続が存在するなら付与してよい |
| mapping_kind | enum, required if mapping_id is present | Mapping の種別: derive / observe / transfer / transform / compose / manual |
| eval_frame_ref | string, optional | 評価フレーム Φ を同定する参照。比較・評価を実行する際に Φ を参照可能にする。注記：Space Metadata として常に付与する必要はないが、compare または check を実行する時点では解決可能でなければならない |
| effect_declaration | object, optional | operation が外部に要求する依存（effect）を宣言する。詳細は後述 |

mapping_kind の列挙は運用で拡張してよいが、compare 相当の意味を持つ拡張は許可しない（compare は 7章 Checks で定義される）。

eval_frame_ref は、比較・評価の実行時に Φ = (ℐ, 𝒜, 𝒞, 𝒪) を同定するためのキーである。
Φ の構成要素（Identity 判定基準、Evidence 採用条件、Condition、Ordering）は、別途参照可能な形で保存されている前提とする。
eval_frame_ref を欠く比較は、Φ の固定を保証できないため、再現可能性が検証不能となる。

#### 6.3.1 effect_declaration（外部依存の宣言）

`effect_declaration` は、operation が外部に要求する依存（effect）を宣言するための任意キーである。
OS層は「参照できること」だけを要求し、具体フィールドや payload 形式は Implementation 層で決定してよい。

##### Minimal shape (normative)

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| effect_id | string | ○ | effect 宣言を参照するための識別子。ID 規則は BSL に準拠（例: `^EF[0-9]{3,}$`） |
| kind | string | ○ | effect の種別（例: toolcall / external_lookup / model_version / dictionary_version / timestamp_asof / heuristic_threshold） |
| ref | string | ○ | 参照対象（tool 名、辞書名、モデル名、外部システム名など） |
| as_of | string | - | 参照時刻または参照版の固定（値の形式は Implementation 層で決定してよい） |
| params | object | - | 閾値、フィルタ、Locale 等の実体。形式は Implementation 層で決定してよい |

##### Interpretation (normative)

- `mapping_kind` は「何をするか」を表す。
- `effect_declaration` は「外部に何を要求するか」を表す。
- 呼出しの記録（action trace）と、外部状態変化の観測参照（effect）は別物であり、相互に含意しない（call≠effect）。
- effect を伴う SSOT commit は、effect 参照に加えて承認参照（approval_ref）および権限根拠（authority_ref）の存在を要件としてよい（詳細は Implementation 層）。
- `effect_declaration` が欠落または参照不能な場合、当該 operation は不成立（undefined）として扱う（分類は 7.6 に従う）。
  - undefined_type: U3（frame_not_closed）
  - diagnostic_reason 例: effect_missing / effect_ref_unresolvable / effect_version_mismatch / permission_denied

#### 6.3.2 Checks logging（pass/failの記録）

pass（通過）と fail（停止）は同一スキーマで記録されなければならない。
fail は停止点（undefined_type: U1〜U4）と diagnostic_reason を含まなければならない。
fail も Evidence Chain へ追記されなければならない。
fail 記録ができない場合は、操作は不成立（U4）として扱う。

NOTE（non-normative）: fail の記録により、同じ失敗の再発を検出可能にし、diagnostic_reason の辞書が育つことで回復手順の入口が明確化される。失敗が資産化され、観測設計の改善につながる。

> 参照：Core DEF-Contract（Contract の定義）、BSL_9 6.5（Contract）

### 6.4 Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| SM-C1 | space_id は比較の許可単位として必須 | Core 9.2.1 |
| SM-C2 | artifact_role は役割の明示として必須 | BSL_1 |
| SM-C3 | view/sidecar には basis_id が必須 | Meaning Identity |
| SM-C4 | evidence には trace_id が必須 | Evidence Continuity |
| SM-C5 | mapping_id がある場合 mapping_kind は必須 | Mapping 明示原則 |

### 6.5 Contract（操作成立の最小契約）

Contract は、特定の operation を特定の Space で成立させるために必要な前提の束である。
本節は Core DEF-Contract が定める最小構成（OperationScope + Φ + RequiredEvidence + EffectDeclaration）を前提とし、BSL における採用手続を伴う operation のために条件付き必須項目を追加する。
Contract は Context の部分射影であり、当該 operation の成立に必要な要素だけを取り出したものである。
Contract は「比較成立」だけでなく「採用（SSOT更新）」の成立にも射程を持つ。

#### 6.5.1 最小構成（normative）

| 構成要素 | 役割 | 必須条件 | 参照 |
|---|---|---|---|
| OperationScope | 操作範囲の宣言 | 常に必須 | BSL_1 4.1 / BSL_7 4.2 |
| Φ（eval_frame_ref） | 評価フレーム | 評価を伴う operation で必須 | Core AX-8 / BSL_7 3.1 |
| RequiredEvidence | 成立に必要な Evidence | evidence 採用・参照で必須 | BSL_4 / BSL_7 6.3, 6.4 |
| EffectDeclaration | 外部依存の宣言 | 外部依存を持つ operation で必須 | 6.3 |
| decision_unit_ref | Design History参照 | 採用を伴う operation で必須 | Core Appendix A.6.3 |
| approval_status | 採用判断の状態 | 採用を伴う operation で必須 | — |
| adoption_ref | 採用根拠の Evidence束 | 採用を伴う operation で必須 | BSL_4 |

NOTE: comparison_scope は OperationScope の部分射影であり、独立要素ではない。

#### 6.5.2 Scope（比較単位と基準の分離）

comparison_scope は「比較と採用を閉じる単位」であり、space_id（Basis層の識別）とは異なる。
comparison_scope は OperationScope の部分射影として扱い、境界が閉じない場合は比較不能として停止する。

#### 6.5.3 Evidence と採用境界（normative）

Evidence は観測結果の記録であり、採用状態を含まない。
採用は Decision Unit（approved）でのみ表現され、approved のみが SSOT 反映の根拠となる。

#### 6.5.4 Decision Unit 参照（decision_unit_ref）

decision_unit_ref は、Design History（Core Appendix A.6.3）の Why/Because/Therefore を束縛する参照である。
Because は adoption_ref を参照し、採用根拠を Evidence に束縛する。

#### 6.5.5 Approval 状態（approval_status）

approval_status は採用判断の状態であり、少なくとも approved/blocked を持つ。
approval_status が不在、または approved 以外の場合、SSOT 更新は不成立として停止する。

approval_status（状態）と approval_ref（承認記録への参照、BSL_4参照）は異なる。
approval_ref は Evidence として記録され、approval_status は Contract で判定される。

#### 6.5.6 Evidence 参照束（minimum）

adoption_ref は、approved が参照する Evidence 束であり、少なくとも trace_id と as-of 束縛を含む。
trace_id を欠く Evidence は採用根拠として不適格である。

判断を受け入れたこと（approved）の Evidence には、結果だけでなく根拠参照が最低限必要である：

| 参照要素 | 役割 | 必須条件 | 参照 |
|---------|------|----------|------|
| approval_ref | 承認の記録 | 必須 | BSL_4 ApprovalRefReading |
| authority_ref | 権限の根拠 | 必須 | BSL_4 AuthorityRefReading |
| trace_id | Evidence Chain の識別 | 必須 | BSL_4 Reading |
| diff_ref | 差分への参照 | 外部作用を伴う更新で必須 | — |
| basis_ref | 基準への参照 | 外部作用を伴う更新で必須 | — |

**定義（normative）**:
- diff_ref: 採用判断の対象となった差分（比較対象）を一意に指す参照
- basis_ref: 判断基準（仕様・規格・合意・試験条件など）を一意に指す参照
- diff_ref/basis_ref は解決可能な参照（resolvable reference）であり、参照先を一意に解決でき、as-of 束縛（版・時刻・コミット等）を付与可能でなければならない。

これらの参照が欠落または不適格な場合、採用は不成立として停止する。

#### 6.5.7 call と effect の区別（ActionTrace≠EffectDelta）

ActionTrace（行為ログ）は「何が実行されたか」の記録であり、EffectDelta（効果差分）ではない。
ActionTrace のみを根拠に採用を主張することはできず、EffectDelta を含む Evidence に束縛されない場合は不成立として停止する。

#### 解釈（normative）

- OperationScope によって禁止される operation は FAIL として扱う（undefined には分類しない）。
- Φ / RequiredEvidence / EffectDeclaration のいずれかが欠落または参照不能な場合、当該 operation は不成立（undefined）として扱う。
- decision_unit_ref / approval_status / adoption_ref が欠落または不適格な場合、採用を伴う operation は不成立（undefined）として扱う。
- 不成立の分類は 7.6 undefined_type に従う。
- Contract は実装パターン（RBAC 等）を規定しない。成立条件の意味論のみを定義する。

> 参照：Core DEF-Contract（Contract の定義）、Core LEM-5（SSOT更新の根拠）、Annex C（Curry–Howard 対応）

---

## 7. Space Boundary Checks（Space 境界チェック）

Checks は、比較・派生・採用の判定結果として FAIL / WARN を返す。
7章は判定条件と停止点を定義し、実行結果の記録形式と追記規則は 6.3.2（Checks logging）が担う。
加えて、不成立（undefined）を第一級の結果型として返してよい（7.6 参照）。
これは既存の FAIL/WARN と互換であり、より詳細な分類を提供する。

NOTE: 本章の各検査表における FAIL は互換表現であり、実装は対応する undefined_type（7.6）を返してよい。

### 7.1 直接比較禁止（No direct compare across spaces）

比較の許可条件は space_id 一致だけでは不十分であり、basis_id 一致を含む。

異なる space_id の成果物を、前提を揃えずに直接比較してはならない。
比較は同一 space_id かつ同一 basis_id の内側でのみ許可される。

例外を設ける場合は、比較前に Mapping を介して前提（basis_id）を整列させ、
整列に用いた mapping_id を明示しなければならない。

#### 検査（最小）

| 検査 | 結果 |
|------|------|
| compare 対象に space_id 不一致がある | FAIL |
| compare 対象に basis_id 不一致がある | FAIL |

### 7.2 SSOT への逆流禁止（No backflow to SSOT）

**原理（normative）**

外部作用を伴う更新は、Evidence（参照束）を伴う append-only 追記としてのみ成立する。
成立しない場合は停止し、その停止も Evidence として記録される。

ある Space の SSOT へ、別 Space から直接書き戻してはならない。
書き戻しが必要な場合は、Mapping として明示し、対応する evidence（trace_id）で説明可能でなければならない。

ssot 更新に関与する経路は、mapping_kind が transfer / transform / manual のいずれかでなければならない。
derive / observe / compose は ssot 更新の根拠として許可されない。
外部作用を伴う ssot 更新は、EffectDeclaration（外部依存の宣言）と承認参照（approval_ref）および権限根拠（authority_ref）を揃えるまで成立させてはならない。

#### append-only とゲートの性質（normative）

SSOT 更新は Evidence Chain への追記に先行してはならない。
追記が成立しない更新は不成立として停止する。
停止も Evidence として追記される（fail ログ）。

この制約により、更新許可の必要条件が「証跡追記の成功」となり、暗黙更新が構造的に禁止される。

#### 検査（最小）

| 検査 | 結果 |
|------|------|
| ssot 更新が mapping_id を欠く | FAIL |
| ssot 更新が evidence（trace_id）参照を欠く | FAIL |
| ssot 更新の mapping_kind が transfer / transform / manual 以外 | FAIL |

### 7.3 Basis drift 検出（Detect basis drift）

同一 space_id 内で basis_id が増殖・漂流する場合、比較の安定性が崩れる。
basis_id が増える場合は、増加理由と影響範囲を evidence として残さなければならない。

#### 検査（最小）

| 検査 | 結果 |
|------|------|
| 同一 space_id で basis_id が増えた変更 | WARN |
| WARN を許容する場合、同一変更内に trace_id を持つ evidence が追加されていない | FAIL |

### 7.4 評価フレーム固定（Evaluation Frame Closure）

比較・評価・最適化の実行時には、評価フレーム Φ = (ℐ, 𝒜, 𝒞, 𝒪) が固定されていなければならない。
Φ が欠落または暗黙に変更された場合、その比較結果は再現不能であり、別評価として扱う。

#### 検査（最小）

| 検査 | 結果 |
|------|------|
| compare 実行時に eval_frame_ref を欠く | FAIL |
| 同一セッション内で Φ が変更された | FAIL（別評価として分割が必要） |
| Φ の構成要素（ℐ, 𝒜, 𝒞, 𝒪）のいずれかが参照不能 | FAIL |

> 参照：Core AX-8（評価フレームの閉包）、BSL_7（Operation）

### 7.5 Cross-Space Operations（Space 跨ぎ操作）

異なる space_id 間での compare は禁止する（7.1 参照）。Cross-Space 操作として route（別 space_id への参照を返す）および redirect（別 space_id の操作へ委譲、権限は継承しない）を許可する。

route/redirect は評価結果を生成しないため、Φ の一致条件を要求しない。

| 操作 | 同一 Space | 異 Space | 許可元 | 備考 |
|------|-----------|----------|--------|------|
| compare | 許可（Φ 一致要） | 禁止 | 7.1, 7.4 | 評価結果を生成 |
| route | - | 許可 | OperationScope | 参照を返す、比較しない |
| redirect | - | 許可 | OperationScope | 操作委譲、権限継承なし |

route/redirect の許可は OperationScope.cross_space_operation で宣言する（BSL_1 4.1 参照）。
compare は OperationScope で許可できない（compare は 7.1 で禁止される）。

#### 検査（最小）

| 検査 | 結果 |
|------|------|
| 異なる space_id 間で compare を実行 | FAIL（SB-C1） |
| route/redirect 時に権限を継承 | FAIL |
| route/redirect が OperationScope.cross_space_operation に含まれない | FAIL |

> 参照：7.1（直接比較禁止）、BSL_1 4.1（Context.OperationScope）

### 7.6 Checks Result Types: undefined_type (normative)

Checks は、比較・派生・採用の判定結果として、不成立（undefined）を第一級の結果型として返してよい。
不成立は次の enum で表す。

#### undefined_type

| 値 | 名称 | 説明 |
|----|------|------|
| U1 | cross_space | space_id が不一致で、直接比較が禁止される |
| U2 | basis_mismatch | basis_id が不一致で、比較の座標系が揃っていない |
| U3 | frame_not_closed | eval_frame_ref が欠落または参照不能で、Φ が閉じていない。Φ の構成要素（ℐ, 𝒜, 𝒞, 𝒪）のいずれかが参照不能な場合も含む。Contract を閉じるために必要な EffectDeclaration（effect_declaration）が欠落または参照不能な場合も含む |
| U4 | evidence_missing | trace_id が欠落、または必要な evidence_chain が欠落し、採用・再現ができない。fail ログを Evidence Chain へ追記できない場合も U4 に分類する（6.3.2 参照） |

#### Mapping from existing FAIL checks (normative)

| 既存検査 | undefined_type |
|----------|---------------|
| space_id mismatch | U1 |
| basis_id mismatch | U2 |
| eval_frame_ref missing/unresolvable or Φ unresolvable | U3 |
| effect_declaration missing/unresolvable | U3 |
| trace_id missing / evidence_chain missing | U4 |

#### CompareResult（BSL_7）との対応 (normative)

CompareResult は BSL_7 が定義する利用者向けの結果型であり、undefined_type は比較不成立時の停止点分類である。

| CompareResult | 典型対応 | 説明 |
|---|---|---|
| MATCH | undefined なし | 比較成立かつ差分なし |
| FAIL | undefined なし | 比較は成立したが差分あり |
| UNDETERMINED | U1 / U2 / U3 | 比較前提が閉じず未確定 |
| EVIDENCE_REQUIRED | U4 | Evidence Chain が不足 |
| PENDING | U3（Ordering 未確定の場合）または なし | 承認・順序・採用判断の未確定 |

#### Optional: recovery_hint (informative)

各 U* は recovery_hint を任意で付与してよい（例: 必要な basis 整列、必要な evidence、必要な effect 参照）。

#### Relation to Semantic Discontinuity (informative)

U1/U2 が回復不能な場合、Core A.4.4 の Semantic Discontinuity に分類される。
U3/U4 は原則として回復可能（欠落要素を補完可能）であり、Discontinuity とは区別される。

#### 拡張方針（normative）

- U1–U4 は OS層で固定し、実装はこの列挙を拡張してはならない。
- より細かい失敗理由が必要な場合は、U* を拡張するのではなく、診断用の別フィールド（例: diagnostic_reason）で表現する。
- U5 以降の追加は BSL 改訂を経る（運用拡張は許可しない）。
- Shell 等の型エラーは、U* への写像を通じて「成立しない理由」を返してよい（詳細は BSL_Shell を参照）。

> 参照：Core DEF-Φ（評価フレームの定義）、Core A.4.4（Semantic Discontinuity）

---

## 8. Interface Definition（OS層 ⇄ 実装層）

### 8.1 OS層が実装層に提供するもの

| 区分 | 内容 |
|------|------|
| 概念定義 | 三軸×三層の意味構造 |
| 制約 | Constraints一覧 |
| ID体系 | 識別子規則 |
| Sidecar原則 | append-only / 非破壊性 |
| 依存方向 | Evidence → Behavior → Flow |

### 8.2 実装層がOS層に準拠する方法

| 方法 | 説明 |
|------|------|
| Slotマッピング | BSL概念をデータに対応付ける |
| Sidecar履歴 | 証拠をappend-onlyで積む |
| View提供 | BSLの構造を破らない操作単位を提供 |
| 一方向依存 | BSL仕様を上書きしない |

---

## 9. Examples（最小例）

### 9.1 Slot（Flow.Part）

```json
{
  "id": "slot_abc123",
  "type": "Flow.Part",
  "payload": {
    "format": "block_reference",
    "block_name": "SENSOR_UNIT_A",
    "layer": "PARTS"
  },
  "evidence_chain": ["R001", "R002"],
  "metadata": {
    "source_file": "assembly.dwg",
    "created_at": "2025-06-01T10:00:00Z"
  }
}
```

その他の Slot 例（Evidence.Reading 等）は BSL Annex（参考）を参照。

---

## 10. ID Scheme（IDスキーム）

Architecture関連のIDは以下の接頭辞を使用する。

| 要素 | 接頭辞 | 例 | 説明 |
|------|--------|-----|------|
| Layer | LY | LY001 | レイヤ定義 |

Slot.id は実装層が自由に決定する（ハッシュ推奨）。BSLはSlotのIDスキームを規定しない。

---

## 11. Extension Points（拡張点）

以下の拡張はBSLの範囲外だが、互換性を損なわない範囲で許容される。

| 拡張 | 説明 | 定義場所 |
|------|------|----------|
| Slot Type Catalog | Slot.type の標準値一覧（参考） | BSL Annex |
| Layer Schema | Layer の参考 JSON Schema | BSL Annex |
| 実装層API | 実装層向けの参考 API 案 | BSL Annex |
| 物理形式マッピング | 各種形式への具体的対応 | Sandboxes |
| UI / 操作体系 | アプリケーション設計 | Sandboxes |
| OSS / 特許戦略 | 公開・権利化の方針 | 別文書 |

---

## 12. Summary（本章のまとめ）

| 項目 | 内容 |
|------|------|
| 対象 | OS層と実装層の境界仕様 |
| 三層モデル | OS / Implementation / Application |
| OS層の責務 | 意味座標の定義と固定 |
| 実装層の責務 | 表現方法の自由な実装 |
| 境界条件 | BD-1〜BD-4 |
| Slot | BSL概念 → 実装データの抽象写像 |
| Space Metadata | space_id / artifact_role / basis_id / trace_id / mapping_id / mapping_kind / effect_declaration |
| Contract | OperationScope / Φ / RequiredEvidence / EffectDeclaration / decision_unit_ref / approval_status / adoption_ref（採用を伴う operation では後者3項目が条件付き必須） |
| Space 境界チェック | 直接比較禁止 / SSOT逆流禁止 / Basis drift検出 / Cross-Space Operations / undefined_type |
| 依存方向 | Application → Implementation → OS（一方向） |
| 不変原則 | append-only / 非侵食 / 意味の固定 |
| IDスキーム | LY（レイヤ） |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | - | 初版 |
| v0.2 | 2025-06 | Core参照ブロック追加、Slot形式化、OSS/特許戦略を別文書に分離 |
| v0.3 | 2025-06 | 統合版。Slot Type CatalogをAppendixへ分離、Layer SchemaをAppendixへ移動、具体ツール名を抽象化 |
| v0.3.1 | 2026-01 | BSL Appendix → BSL Annex 参照名変更 |
| v0.4 | 2026-01 | Space Metadata（6章）、Space Boundary Checks（7章）を追加。章番号再編 |
| v0.5 | 2026-01 | 評価フレーム Φ を追加（3.1 固定するもの、6.3 eval_frame_ref、7.4 評価フレーム固定）。Core AX-8 との接続 |
| v0.6 | 2026-01-14 | Cross-Space Operations（7.5）を追加。route/redirect を定義。OperationScope との関係を明記。mapping_kind 拡張注記を追加 |
| v0.7 | 2026-01 | effect_declaration を 6.3 に追加。undefined_type（U1〜U4）を 7.6 に追加。7章冒頭に FAIL/WARN との互換注記を追加。Core DEF-Φ / DEF-Contract との接続を明記 |
| v0.8 | 2026-01-20 | Contract（6.5）を新設。undefined_type に拡張方針を追記。6.3 参照を更新 |
| v0.9 | 2026-01 | call≠effect 原則を追加（6.3）。SSOT commit の approval/authority 要件を追加（7.2） |
| v1.0 | 2026-01 | Contract（6.5）を拡張：decision_unit_ref/approval_status/adoption_ref を追加。comparison_scope を定義。ActionTrace≠EffectDelta を明文化。Core LEM-5 との接続を追加 |
| v1.1 | 2026-01 | append-only のゲート性を規範化（7.2）。根拠参照の最低要件を網羅的に定義（6.5.6：diff_ref/basis_ref追加）。fail 記録の資産化を規範化（6.3）。最上位原理を 7.2 冒頭に追加。6.3 見出し階層調整（6.3.1 effect_declaration, 6.3.2 Checks logging）。diff_ref/basis_ref に参照型制約を追加 |
| v1.1.1 | 2026-03 | Annex 参照文言を informative 補助に統一。BSL_0 の Informative 宣言との整合 |
| v1.1.2 | 2026-03 | 公開前の定義整合および用語整合パッチを適用 |
| v1.1.3 | 2026-03 | §6.5.1 Contract 表の BSL_7 参照先を実在する節番号に修正（OperationScope→4.2、Φ→3.1、RequiredEvidence→6.3/6.4） |