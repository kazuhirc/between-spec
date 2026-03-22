# BSL_7. Operation 仕様

**Version: v0.5.2**

---

## Core Dependency

本章が依拠するCoreの定義を以下に示す。

| 参照先 | Core節 | 本章での役割 |
|--------|--------|-------------|
| 依存ポリシー（軸間） | A.3.1 | Evidence → Behavior → Flow の一方向性 |
| Sidecar | A.5.2 | 運転記録の非破壊的蓄積 |
| View | A.5.1 | 条件を与えて意味を読み取る仕組み |
| 外側レイヤと三軸の関係 | A.6.2 | Operation は三軸を参照するが変更しない |
| 評価フレームの閉包 | AX-8 | 比較・評価の成立条件として Φ を固定 |

BSL独自の定義：OperationRecord / State / Transition / Session

---

## 1. Purpose（この章の目的）

本章は、Operation（運転モデル）を機械可読なデータ構造として仕様化する。

Operationは、三軸（Flow / Behavior / Evidence）および外側レイヤ（Variant / Design History）で定義された意味構造を、実際の運用で実行・記録するための仕組みである。

仕様化の範囲：
- OperationRecord / State / Transition のデータモデル
- 運転セッションの構造
- 読み取り（Read）・適用（Apply）・記録（Record）の操作定義
- 三軸・外側レイヤとの依存関係

仕様化の範囲外：
- 具体的なドメイン固有の運転手順 → Sandboxes
- UI/制御系の実装詳細 → Sandboxes

---

## 2. Operation の位置づけ

### 2.1 実行レイヤとしての役割

```
┌─────────────────────────────────────────┐
│            外側レイヤ                    │
│   Context / Variant / Design History    │
└───────────────┬─────────────────────────┘
                │ 参照
                ▼
┌─────────────────────────────────────────┐
│           三軸（SSOT）                   │
│   Flow ←── Behavior ←── Evidence        │
└───────────────┬─────────────────────────┘
                │ 参照（読み取りのみ）
                ▼
┌─────────────────────────────────────────┐
│         Operation（実行レイヤ）          │
│   ★本章                                 │
│   State / Transition / Record           │
└───────────────┬─────────────────────────┘
                │ 追記
                ▼
┌─────────────────────────────────────────┐
│         Sidecar（運転記録）              │
│   Evidence Chain への追加               │
└─────────────────────────────────────────┘
```

### 2.2 Operation が扱う6つの領域

| 領域 | 説明 | 対応する三軸・外側レイヤ |
|------|------|-------------------------|
| State | 運転時の状態 | Flow / Variant に基づく |
| Transition | 状態遷移 | Behavior (Step/Event) に基づく |
| Read | 観測値の読み取り | Evidence (Reading/Condition) を参照 |
| Apply | 設定の適用 | Variant の選択結果を参照 |
| Record | 運転記録 | Evidence Chain への追記 |
| Explain | 説明可能性 | Design History を参照 |

注記：
- 本章でいう Operation は、運転上の読み取り・適用・記録を扱う実行レイヤである
- SSOT commit は Operation そのものではなく、差分参照と承認参照に基づく外側手続として別に成立する
- Operation は commit の前提となる action trace、ordering、approval や authority 参照を記録し得るが、採用確定そのものを代替しない

---

## 3. Data Model（データモデル）

### 3.1 OperationSession（運転セッション）

一連の運転を束ねるセッション構造。

#### Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^OS[0-9]{3,}$",
      "description": "Session ID（例：OS001）"
    },
    "target_flow_id": {
      "type": "string",
      "pattern": "^[PAL][0-9]{3,}$",
      "description": "対象Flow要素のID"
    },
    "target_sequence_id": {
      "type": "string",
      "pattern": "^Q[0-9]{3,}$",
      "description": "実行するSequence ID"
    },
    "variant_selections": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "variant_id": { "type": "string", "pattern": "^V[0-9]{3,}$" },
          "selected_option_id": { "type": "string", "pattern": "^VO[0-9]{3,}$" }
        },
        "required": ["variant_id", "selected_option_id"]
      },
      "description": "採用済みVariant選択"
    },
    "start_time": {
      "type": "string",
      "format": "date-time",
      "description": "セッション開始時刻"
    },
    "end_time": {
      "type": "string",
      "format": "date-time",
      "description": "セッション終了時刻"
    },
    "status": {
      "type": "string",
      "enum": ["pending", "running", "completed", "aborted"],
      "default": "pending"
    },
    "records": {
      "type": "array",
      "items": { "$ref": "#/$defs/OperationRecord" },
      "description": "運転記録一覧"
    },
    "eval_frame_snapshot": {
      "type": "object",
      "properties": {
        "identity_criteria_ref": { "type": "string", "description": "Identity 判定基準 ℐ への参照" },
        "evidence_adoption_ref": { "type": "string", "description": "Evidence 採用条件 𝒜 への参照" },
        "condition_ref": { "type": "string", "description": "Condition 𝒞 への参照" },
        "ordering_ref": { "type": "string", "description": "Ordering 𝒪 への参照" }
      },
      "description": "評価フレーム Φ のスナップショット。セッション開始時に固定"
    }
  },
  "required": ["id", "target_sequence_id"]
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^OS[0-9]{3,}$` | プロジェクト内で一意 |
| target_flow_id | string | - | Flow ID参照 | 対象Flow要素 |
| target_sequence_id | string | ○ | Sequence ID参照 | 実行するSequence |
| variant_selections | array | - | - | 採用済みVariant選択 |
| start_time | string | - | ISO 8601 | 開始時刻 |
| end_time | string | - | ISO 8601 | 終了時刻 |
| status | string | - | enum | セッション状態 |
| records | array | - | - | 運転記録一覧 |
| eval_frame_snapshot | object | - | - | 評価フレーム Φ のスナップショット |

eval_frame_snapshot は、セッション開始時に評価フレーム Φ = (ℐ, 𝒜, 𝒞, 𝒪) を固定するために使用する。
同一セッション内で Φ を暗黙に変更してはならない。Φ を変更する場合は、セッションを分割するか、別評価として履歴化する。

---

### 3.2 OperationRecord（運転記録）

運転中の個々の記録。

#### Schema

```json
{
  "$defs": {
    "OperationRecord": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^OR[0-9]{3,}$",
          "description": "Record ID（例：OR001）"
        },
        "timestamp": {
          "type": "string",
          "format": "date-time",
          "description": "記録時刻"
        },
        "step_id": {
          "type": "string",
          "pattern": "^S[0-9]{3,}$",
          "description": "実行中のStep ID"
        },
        "event_id": {
          "type": "string",
          "pattern": "^E[0-9]{3,}$",
          "description": "発生したEvent ID"
        },
        "state": {
          "$ref": "#/$defs/OperationState",
          "description": "記録時点の状態"
        },
        "transition": {
          "$ref": "#/$defs/Transition",
          "description": "状態遷移（あれば）"
        },
        "readings": {
          "type": "array",
          "items": {
            "type": "string",
            "pattern": "^R[0-9]{3,}$"
          },
          "description": "取得したReading ID一覧"
        },
        "record_type": {
          "type": "string",
          "enum": ["read", "apply", "transition", "checkpoint", "error"],
          "description": "記録種別"
        }
      },
      "required": ["id", "timestamp", "record_type"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^OR[0-9]{3,}$` | セッション内で一意 |
| timestamp | string | ○ | ISO 8601 | 記録時刻 |
| step_id | string | - | Step ID参照 | 実行中のStep |
| event_id | string | - | Event ID参照 | 発生したEvent |
| state | OperationState | - | - | 記録時点の状態 |
| transition | Transition | - | - | 状態遷移 |
| readings | array | - | Reading ID参照 | 取得したReading |
| record_type | string | ○ | enum | 記録種別 |

---

### 3.3 OperationState（運転状態）

運転時点の状態を表す。

#### Schema

```json
{
  "$defs": {
    "OperationState": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^OT[0-9]{3,}$",
          "description": "State ID（例：OT001）"
        },
        "flow_snapshot": {
          "type": "object",
          "description": "Flow要素の状態スナップショット"
        },
        "variant_state": {
          "type": "object",
          "description": "適用中のVariant状態"
        },
        "condition_snapshot": {
          "type": "object",
          "description": "現在のCondition状態"
        },
        "step_status": {
          "type": "string",
          "enum": ["not_started", "in_progress", "completed", "skipped", "error"],
          "description": "Stepの進行状態"
        }
      },
      "required": ["id"]
    }
  }
}
```

---

### 3.4 Transition（状態遷移）

状態間の遷移を表す。

#### Schema

```json
{
  "$defs": {
    "Transition": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^OX[0-9]{3,}$",
          "description": "Transition ID（例：OX001）"
        },
        "from_state_id": {
          "type": "string",
          "pattern": "^OT[0-9]{3,}$",
          "description": "遷移元State ID"
        },
        "to_state_id": {
          "type": "string",
          "pattern": "^OT[0-9]{3,}$",
          "description": "遷移先State ID"
        },
        "trigger_event_id": {
          "type": "string",
          "pattern": "^E[0-9]{3,}$",
          "description": "遷移を引き起こしたEvent ID"
        },
        "trigger_reading_id": {
          "type": "string",
          "pattern": "^R[0-9]{3,}$",
          "description": "遷移を引き起こしたReading ID"
        }
      },
      "required": ["id", "from_state_id", "to_state_id"]
    }
  }
}
```

#### confirmation_event（参考）

提案と採用の境界（confirmation_event）を Transition として履歴化する場合、以下のフィールドを追加してよい。

| フィールド | 型 | 説明 |
|-----------|-----|------|
| confirmation_type | enum | propose_to_adopt / reject / escalate |
| scope_ref | string | 参照した OperationScope の scope_id。埋め込みではなく参照とし、同一 scope_id を再利用可能にする |
| approver_ref | string | 承認者の参照（人またはシステム） |

これにより、同じ出力でも「誰がどの権限で採用したか」を追跡できる。

注記: scope_ref は OperationScope を参照するための ID であり、OperationScope の内容を複製しない。

---

### 3.5 Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| OS-C1 | Session ID はプロジェクト内で一意 | 同一性判定の前提 |
| OS-C2 | target_sequence_id は既存Sequence ID | 参照整合性 |
| OS-C3 | variant_selections は Design History で採択済みのもの | 設計と運転の分離 |
| OR-C1 | Record は append-only | 非破壊性（Core A.5.2） |
| OR-C2 | Record は Flow / Behavior / Variant を変更しない | 依存ポリシー |
| OR-C3 | readings は Evidence Chain に追加される | Sidecar蓄積 |
| OT-C1 | flow_snapshot は参照のみ（変更禁止） | 非破壊性 |
| OX-C1 | Transition は Behavior の Step/Event に対応 | 整合性検証 |

---

## 4. Dependency Policy（依存ポリシー）

### 4.1 Operation と三軸の関係

Operation は三軸を参照するが、三軸を変更しない。

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| Operation | Flow | ○（読取） | State の計算元として |
| Operation | Behavior | ○（読取） | Transition のテンプレートとして |
| Operation | Evidence | ○（読取／追記） | Read で参照、Record で Evidence Chain に append-only |
| Flow | Operation | × | 禁止 |
| Behavior | Operation | × | 禁止 |

### 4.2 Operation と外側レイヤの関係

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| Operation | Variant | ○（読取） | 採択済み選択を参照 |
| Operation | Design History | ○（読取） | Explain のため |
| Operation | Context | ○（読取） | 条件判定のため |
| Variant | Operation | × | 禁止 |
| Design History | Operation | × | 禁止 |

#### 承認記録の位置づけ

承認は Operation の Event/Transition として記録できるが、承認参照（ApprovalRefReading）は許可や成立を含意しない。許可判定は Context（OperationScope を含む）に属し、Operation はその結果を記録する。

approval_ref は、operation の実行に対する「承認の記録」を参照するためのフィールドである。
authority_ref は、「権限の根拠」（例：role/policy/委任/署名）を参照するためのフィールドである。
承認（approval_ref）は、主体（actor_ref）が権限根拠（authority_ref）の下で実行したことを、外部手続きとして記録する。

外部作用を伴う operation では、呼出し記録（action trace）と結果観測参照（effect）を分離して記録する。
承認参照のみで「実行済み／反映済み」を含意してはならない（採用は Design History に閉じる）。

| 要素 | 記録場所 | 備考 |
|------|----------|------|
| 権限根拠 | Evidence（AuthorityRefReading） | 権限の根拠を参照（真偽判定はしない） |
| 承認参照 | Evidence（ApprovalRefReading） | 承認の記録を参照（許可を含意しない） |
| 許可判定 | Context（OperationScope） | 成立判定・実行許可 |
| 承認イベント | Operation（Event/Transition） | 結果の履歴化 |
| 採用確定 | Design History（Therefore.status=approved） | 判断の確定 |

注記（approve と adopt の関係）: 承認イベント（approve/confirmation）は Operation に記録され、採用（adopt）は Design History の Therefore.status=approved として記録される。承認イベントが記録されても、Therefore.status=approved が存在しなければ採用は成立しない。


### 4.3 Sidecar との関係

Operation の出力は Sidecar（Evidence Chain）に追記される。

| 項目 | 説明 |
|------|------|
| 追記対象 | Reading / Condition / Ordering |
| 追記ルール | append-only |
| 参照元 | OperationRecord.readings |

---

## 5. Operation Tenets（運用原則）

| 原則 | ID | 説明 |
|------|-----|------|
| Non-destructive | OT-1 | Flow / Variant を直接変更しない |
| Evidence-first | OT-2 | すべての運転判断は Evidence に基づく |
| Traceable | OT-3 | Operation → Evidence → Design History へ遡及可能 |
| Deterministic | OT-4 | 同じ入力（Flow/Behavior/Variant/Evidence）に対し同じ結果 |
| Append-only | OT-5 | 記録は Sidecar にのみ追記 |
| Explainable | OT-6 | Operation は Design History を参照して説明可能 |
| Frame-fixed | OT-7 | 同一セッション内で評価フレーム Φ を暗黙に変更しない。変更する場合はセッション分割または別評価として履歴化 |
| Scope-aware | OT-8 | 操作を記録する Event/Transition は scope_ref を持つ。許可されない操作は execute として記録せず、propose または reject として記録する |

---

## 6. Operations（操作）

### 6.1 Session 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_session | target_sequence_id | OperationSession | id 自動採番 |
| start_session | session_id | OperationSession | status: running に変更 |
| complete_session | session_id | OperationSession | status: completed に変更 |
| abort_session | session_id, reason | OperationSession | status: aborted に変更 |

### 6.2 Record 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| record_read | session_id, reading_id | OperationRecord | Evidence Chain に追記 |
| record_transition | session_id, from_state, to_state, trigger | OperationRecord | Behavior 整合性チェック |
| record_checkpoint | session_id, state | OperationRecord | 状態スナップショット保存 |
| record_error | session_id, error_info | OperationRecord | エラー情報記録 |

### 6.3 Query 操作

| 操作 | 入力 | 出力 | 説明 |
|------|------|------|------|
| get_current_state | session_id | OperationState | 現在の状態を取得 |
| get_session_history | session_id | OperationRecord[] | セッションの記録一覧 |
| explain_state | session_id, state_id | DesignHistory[] | 状態の理由を Design History から取得 |
| verify_sequence | session_id | boolean | Behavior との整合性を検証 |
| compare | session_id, target_a, target_b | CompareResult | 同一性を判定（Φ 必須） |

compare 操作は、eval_frame_snapshot（評価フレーム Φ）が固定されている場合にのみ実行可能である。
Φ を欠く compare は UNDETERMINED を返す。Flow/Behavior 章の Compare は本操作を正規の入口として参照する。

### 6.4 CompareResult（比較結果）

Compare の結果は次の5種に分類される。

| output | meaning | next_action |
|--------|---------|-------------|
| `MATCH` | Φ 固定済みで一致 | 次工程へ進む |
| `FAIL` | Φ 固定済みで不一致 | 不一致として扱う（diff 参照） |
| `UNDETERMINED` | Φ 固定不能（Semantic Discontinuity） | Φ 要素（ℐ, 𝒜, 𝒞, 𝒪）の明示要求 |
| `EVIDENCE_REQUIRED` | 𝒜 不足（必要 Reading 不足） | 追加 Evidence 要求 |
| `PENDING` | 𝒪 未合意（順序未確定） | 順序合意手順を提示 |

注：`FAIL` は「不一致」、`UNDETERMINED` は「比較不成立」に意味を限定する。

#### CompareResult Schema

```json
{
  "$defs": {
    "CompareResult": {
      "type": "object",
      "properties": {
        "output": {
          "type": "string",
          "enum": ["MATCH", "FAIL", "UNDETERMINED", "EVIDENCE_REQUIRED", "PENDING"],
          "description": "比較結果"
        },
        "eval_frame_snapshot": {
          "type": ["string", "null"],
          "description": "評価フレーム Φ のスナップショット参照（Core A.4.5）"
        },
        "diff_ref": {
          "type": "string",
          "description": "FAIL 時の差分参照"
        },
        "missing": {
          "type": "array",
          "items": { "type": "string" },
          "description": "EVIDENCE_REQUIRED 時の不足 Reading ID"
        },
        "pending": {
          "type": "string",
          "description": "PENDING 時の未合意項目"
        }
      },
      "required": ["output"]
    }
  }
}
```

#### CompareResult と undefined_type の対応

CompareResult は利用者向けの結果型であり、BSL_9 §7.6 の undefined_type は比較不成立時の停止点分類である。

| CompareResult | 典型対応 | 説明 |
|---|---|---|
| MATCH | undefined なし | 比較成立かつ差分なし |
| FAIL | undefined なし | 比較は成立したが差分あり |
| UNDETERMINED | U1 / U2 / U3 | 比較前提が閉じず未確定 |
| EVIDENCE_REQUIRED | U4 | Evidence Chain が不足 |
| PENDING | U3（Ordering 未確定の場合）または なし | 承認・順序・採用判断の未確定 |
#### CompareResult Examples

```json
{ "output": "MATCH", "eval_frame_snapshot": "EF001" }
{ "output": "FAIL", "diff_ref": "D123", "eval_frame_snapshot": "EF001" }
{ "output": "UNDETERMINED", "eval_frame_snapshot": null }
{ "output": "EVIDENCE_REQUIRED", "missing": ["R099", "R102"] }
{ "output": "PENDING", "pending": "ordering" }
```

---

## 7. Examples（最小例）

本節の例は BSL_1 第10章「Running Example」で定義された共通例に基づく。

### 7.1 最小Session

```json
{
  "id": "OS001",
  "target_flow_id": "A001",
  "target_sequence_id": "Q001",
  "variant_selections": [
    { "variant_id": "V001", "selected_option_id": "VO001" }
  ],
  "start_time": "2025-06-01T10:00:00Z",
  "status": "running",
  "records": []
}
```

### 7.2 OperationRecord（Read）

```json
{
  "id": "OR001",
  "timestamp": "2025-06-01T10:05:00Z",
  "step_id": "S001",
  "record_type": "read",
  "readings": ["R001"],
  "state": {
    "id": "OT001",
    "step_status": "in_progress"
  }
}
```

### 7.3 OperationRecord（Transition）

```json
{
  "id": "OR002",
  "timestamp": "2025-06-01T10:10:00Z",
  "step_id": "S001",
  "event_id": "E001",
  "record_type": "transition",
  "transition": {
    "id": "OX001",
    "from_state_id": "OT001",
    "to_state_id": "OT002",
    "trigger_event_id": "E001"
  }
}
```

### 7.4 完了したSession

```json
{
  "id": "OS001",
  "target_flow_id": "A001",
  "target_sequence_id": "Q001",
  "variant_selections": [
    { "variant_id": "V001", "selected_option_id": "VO001" }
  ],
  "start_time": "2025-06-01T10:00:00Z",
  "end_time": "2025-06-01T10:30:00Z",
  "status": "completed",
  "records": [
    { "id": "OR001", "record_type": "read", "timestamp": "2025-06-01T10:05:00Z" },
    { "id": "OR002", "record_type": "transition", "timestamp": "2025-06-01T10:10:00Z" },
    { "id": "OR003", "record_type": "checkpoint", "timestamp": "2025-06-01T10:30:00Z" }
  ]
}
```

---

## 8. ID Scheme（IDスキーム）

Operation関連のIDは以下の接頭辞を使用する。

| 要素 | 接頭辞 | 例 | 説明 |
|------|--------|-----|------|
| OperationSession | OS | OS001 | 運転セッション |
| OperationRecord | OR | OR001 | 運転記録 |
| OperationState | OT | OT001 | 運転状態 |
| Transition | OX | OX001 | 状態遷移 |

三軸・外側レイヤのIDスキームとの衝突を避けるため、Operation関連はすべて「O」で始まる（ただしOrdering: O との区別のため2文字目で識別）。

---

## 9. Extension Points（拡張点）

以下の拡張はBSLの範囲外だが、互換性を損なわない範囲で許容される。

| 拡張 | 説明 | 定義場所 |
|------|------|----------|
| 自動運転 | Behavior に基づく自動実行 | Sandboxes |
| 並列実行 | 複数Session の同時実行 | Sandboxes |
| ロールバック | 状態の巻き戻し（読取専用） | Sandboxes |
| リアルタイム監視 | State の継続的監視 | Sandboxes |

---

## 10. Summary（本章のまとめ）

| 項目 | 内容 |
|------|------|
| 対象 | Operation（運転モデル） |
| 位置づけ | 実行レイヤ（三軸・外側レイヤの下位） |
| 主要構造 | Session / Record / State / Transition |
| 三軸との関係 | 参照のみ（変更禁止） |
| 出力先 | Sidecar（Evidence Chain）への追記 |
| 運用原則 | Non-destructive / Evidence-first / Traceable / Deterministic / Append-only / Explainable / Frame-fixed / Scope-aware |
| IDスキーム | OS / OR / OT / OX |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | - | 初版（思想説明を含む旧版） |
| v0.2 | 2025-06 | Core参照ブロック追加、JSON Schema形式化、IDスキーム整理、思想成分排除 |
| v0.3 | 2026-01 | eval_frame_snapshot 追加、OT-7（Frame-fixed）追加、compare 操作追加。Core AX-8 との接続 |
| v0.3.1 | 2026-01-14 | 承認記録の位置づけを追記。confirmation_event 参考定義追加。OT-8（Scope-aware）追加。approve/adopt 関係を注記 |
| v0.4 | 2026-01-15 | CompareResult を5種分類（MATCH/FAIL/UNDETERMINED/EVIDENCE_REQUIRED/PENDING）に拡張。CompareResult Schema 追加（沈黙パターン対応） |
| v0.5 | 2026-01 | authority_ref と approval_ref を分離。外部作用の呼出し記録と結果観測の分離を追記 |
| v0.5.1 | 2026-03 | 公開前整合パッチ：§2.2 に Operation と SSOT commit の境界に関する注記を追加。§10 Summary 運用原則に Frame-fixed を追加 |
| v0.5.2 | 2026-03 | 公開前整合パッチ：Running Example 参照を第10章に統一。CompareResult と undefined_type の対応表を §6.4 に追加 |
