# BSL_8. Continuity 仕様

**Version: v0.3.1**

---

## Core Dependency

本章が依拠するCoreの定義を以下に示す。

| 参照先 | Core節 | 本章での役割 |
|--------|--------|-------------|
| Sidecar | A.5.2 | append-only による非破壊的蓄積 |
| 依存ポリシー | A.3 | 参照関係の一方向性 |
| Meaning Identity | A.4 | 再開・復元時の同一性判定 |
| Design History | A.6.3 | 判断の連鎖の保持 |

BSL独自の定義：ContinuityRecord / ResumePoint / RestorePoint / Checkpoint

---

## 1. Purpose（この章の目的）

本章は、Continuity（継続性）を機械可読なデータ構造として仕様化する。

Continuityは、三軸（Flow / Behavior / Evidence）、外側レイヤ（Variant / Design History）、および Operation の「時系列的な連続性」を保証する仕組みである。

仕様化の範囲：
- ContinuityRecord / ResumePoint / RestorePoint のデータモデル
- Checkpoint の構造と作成ルール
- 再開・復元の操作定義
- 各章の構造との参照関係

仕様化の範囲外：
- 具体的な技能伝承プロセスの実装 → Sandboxes
- 組織運営・承認フローの詳細 → Sandboxes

---

## 2. Continuity の位置づけ

### 2.1 連続性保証レイヤとしての役割

```
┌─────────────────────────────────────────┐
│       Continuity（連続性保証）           │
│       ★本章                             │
│   History / Chain / Anchor              │
└───────────────┬─────────────────────────┘
                │ 参照・束ねる
                ▼
┌─────────────────────────────────────────┐
│         Operation（実行レイヤ）          │
│   Session / Record / State              │
└───────────────┬─────────────────────────┘
                │
┌───────────────┼─────────────────────────┐
│            外側レイヤ                    │
│   Context / Variant / Design History    │
└───────────────┬─────────────────────────┘
                │
┌───────────────┼─────────────────────────┐
│           三軸（SSOT）                   │
│   Flow ←── Behavior ←── Evidence        │
└─────────────────────────────────────────┘
```

### 2.2 Continuity が扱う3つの要素

| 要素 | 説明 | 対応するBSL章 |
|------|------|---------------|
| History | 判断・実行の時間軸 | Design History (BSL_6) / Operation (BSL_7) |
| Chain | 観測値の連続性 | Evidence Chain (BSL_4) |
| Anchor | 復元可能性の基準 | Flow / Variant / Ordering |

### 2.3 継続性の最小単位

継続性設計の最小単位は「個人（利用者単位）」である。

| 原則 | 説明 |
|------|------|
| 最小単位 | 技能伝承の最小単位は組織ではなく個人 |
| 自己参照可能性 | 将来の自分が読み取れる形式で記述する |
| 他者への波及 | 自己参照可能な記述は、他者にも伝達可能である |

---

## 3. Data Model（データモデル）

### 3.1 ContinuityRecord（継続性レコード）

継続性を担保するための参照集合。

#### Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^CT[0-9]{3,}$",
      "description": "Continuity Record ID（例：CT001）"
    },
    "flow_ref": {
      "type": "string",
      "pattern": "^[PAL][0-9]{3,}$",
      "description": "参照するFlow要素ID"
    },
    "behavior_ref": {
      "type": "string",
      "pattern": "^[ESQ][0-9]{3,}$",
      "description": "参照するBehavior要素ID"
    },
    "variant_ref": {
      "type": "string",
      "pattern": "^V[0-9]{3,}$",
      "description": "参照するVariant ID"
    },
    "operation_ref": {
      "type": "string",
      "pattern": "^OS[0-9]{3,}$",
      "description": "参照するOperation Session ID"
    },
    "evidence_chain": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^[RCO][0-9]{3,}$"
      },
      "description": "Evidence Chain（Reading/Condition/Ordering ID一覧）"
    },
    "design_history_ref": {
      "type": "string",
      "pattern": "^DH[0-9]{3,}$",
      "description": "参照するDesign History ID"
    },
    "anchor": {
      "$ref": "#/$defs/Anchor",
      "description": "復元可能性の基準"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "作成時刻"
    },
    "description": {
      "type": "string",
      "description": "継続性レコードの説明"
    }
  },
  "required": ["id", "anchor"]
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^CT[0-9]{3,}$` | プロジェクト内で一意 |
| flow_ref | string | - | Flow ID参照 | 参照するFlow要素 |
| behavior_ref | string | - | Behavior ID参照 | 参照するBehavior要素 |
| variant_ref | string | - | Variant ID参照 | 参照するVariant |
| operation_ref | string | - | Session ID参照 | 参照するOperation |
| evidence_chain | array | - | Evidence ID参照 | Evidence Chain |
| design_history_ref | string | - | DH ID参照 | 参照するDesign History |
| anchor | Anchor | ○ | - | 復元可能性の基準 |
| timestamp | string | - | ISO 8601 | 作成時刻 |
| description | string | - | - | 説明 |

---

### 3.2 Anchor（復元基準）

復元可能性を保証する基準点。

#### Schema

```json
{
  "$defs": {
    "Anchor": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": ["flow", "behavior", "evidence", "variant"],
          "description": "基準の種別"
        },
        "target_id": {
          "type": "string",
          "description": "基準となる要素のID"
        },
        "ordering_ref": {
          "type": "string",
          "pattern": "^O[0-9]{3,}$",
          "description": "Ordering ID（時系列上の位置）"
        }
      },
      "required": ["type", "target_id"]
    }
  }
}
```

---

### 3.3 ResumePoint（再開点）

「どこから再開できるか」を示す論理点。

#### Schema

```json
{
  "$defs": {
    "ResumePoint": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^CR[0-9]{3,}$",
          "description": "Resume Point ID（例：CR001）"
        },
        "continuity_record_id": {
          "type": "string",
          "pattern": "^CT[0-9]{3,}$",
          "description": "対応するContinuity Record ID"
        },
        "step_id": {
          "type": "string",
          "pattern": "^S[0-9]{3,}$",
          "description": "再開すべきStep ID"
        },
        "state_snapshot": {
          "type": "object",
          "description": "再開時の状態スナップショット"
        },
        "preconditions": {
          "type": "array",
          "items": { "type": "string" },
          "description": "再開に必要な前提条件"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "required": ["id", "continuity_record_id", "step_id"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^CR[0-9]{3,}$` | プロジェクト内で一意 |
| continuity_record_id | string | ○ | CT ID参照 | 対応するContinuity Record |
| step_id | string | ○ | Step ID参照 | 再開すべきStep |
| state_snapshot | object | - | - | 再開時の状態 |
| preconditions | array | - | - | 再開に必要な前提条件 |
| created_at | string | - | ISO 8601 | 作成時刻 |

---

### 3.4 RestorePoint（復元点）

「どこに戻ればよいか」を示す判断点。

#### Schema

```json
{
  "$defs": {
    "RestorePoint": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^CX[0-9]{3,}$",
          "description": "Restore Point ID（例：CX001）"
        },
        "continuity_record_id": {
          "type": "string",
          "pattern": "^CT[0-9]{3,}$",
          "description": "対応するContinuity Record ID"
        },
        "design_history_id": {
          "type": "string",
          "pattern": "^DH[0-9]{3,}$",
          "description": "復元対象のDesign History ID"
        },
        "reason": {
          "type": "string",
          "description": "復元が必要な理由"
        },
        "restore_scope": {
          "type": "string",
          "enum": ["flow", "behavior", "variant", "full"],
          "description": "復元の範囲"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "required": ["id", "continuity_record_id", "design_history_id"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^CX[0-9]{3,}$` | プロジェクト内で一意 |
| continuity_record_id | string | ○ | CT ID参照 | 対応するContinuity Record |
| design_history_id | string | ○ | DH ID参照 | 復元対象のDesign History |
| reason | string | - | - | 復元が必要な理由 |
| restore_scope | string | - | enum | 復元の範囲 |
| created_at | string | - | ISO 8601 | 作成時刻 |

---

### 3.5 Checkpoint（チェックポイント）

定期的または重要時点での継続性スナップショット。

#### Schema

```json
{
  "$defs": {
    "Checkpoint": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^CP[0-9]{3,}$",
          "description": "Checkpoint ID（例：CP001）"
        },
        "continuity_record_id": {
          "type": "string",
          "pattern": "^CT[0-9]{3,}$",
          "description": "対応するContinuity Record ID"
        },
        "checkpoint_type": {
          "type": "string",
          "enum": ["milestone", "periodic", "manual", "error_recovery"],
          "description": "チェックポイント種別"
        },
        "label": {
          "type": "string",
          "description": "チェックポイントのラベル"
        },
        "resume_point_id": {
          "type": "string",
          "pattern": "^CR[0-9]{3,}$",
          "description": "関連するResume Point ID"
        },
        "restore_point_id": {
          "type": "string",
          "pattern": "^CX[0-9]{3,}$",
          "description": "関連するRestore Point ID"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "required": ["id", "continuity_record_id", "checkpoint_type"]
    }
  }
}
```

---

### 3.6 Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| CT-C1 | ContinuityRecord は append-only | 非破壊性（Core A.5.2） |
| CT-C2 | anchor は必須（復元可能性の保証） | 継続性の定義 |
| CT-C3 | 参照先は既存の要素であること | 参照整合性 |
| CT-C4 | evidence_chain は Evidence Chain（BSL_4）の append-only 原則に従う | 観測の連続性 |
| CR-C1 | ResumePoint は step_id を持つこと | 再開位置の明確化 |
| CX-C1 | RestorePoint は design_history_id を持つこと | 判断の復元 |
| CP-C1 | Checkpoint は checkpoint_type を持つこと | 分類の明確化 |

---

## 4. Continuity Principles（継続性の原則）

| 原則 | ID | 説明 |
|------|-----|------|
| Append-only | CP-1 | Sidecar（Evidence/History）は追記専用、過去は書き換えない |
| Two-layer History | CP-2 | 設計履歴（Design History）と運転履歴（Operation）を対で保持 |
| Separation | CP-3 | 理由（Design History）と動作（Operation/Evidence）を混同しない |
| Resumability | CP-4 | どこからでも再開できる構造を維持 |
| Restorability | CP-5 | どこに戻ればよいかが明確な構造を維持 |
| Closure | CP-6 | 比較・評価は、同一 SSOT を参照し、Sidecar（Basis/Condition/Ordering）と投影規則が固定され参照可能である場合にのみ成立する（改訂は version として記録する） |

---

## 5. Dependency Policy（依存ポリシー）

### 5.1 Continuity と他レイヤの関係

Continuity は全レイヤを参照するが、変更しない。

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| Continuity | Flow | ○（読取） | Anchor / 状態スナップショットとして |
| Continuity | Behavior | ○（読取） | ResumePoint の step_id として |
| Continuity | Evidence | ○（読取） | Evidence Chain として |
| Continuity | Variant | ○（読取） | variant_ref として |
| Continuity | Design History | ○（読取） | RestorePoint として |
| Continuity | Operation | ○（読取） | operation_ref として |
| 各レイヤ | Continuity | × | 禁止（逆流防止） |

---

## 6. Operations（操作）

### 6.1 ContinuityRecord 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_continuity_record | anchor | ContinuityRecord | id 自動採番 |
| add_evidence_to_chain | ct_id, evidence_id | ContinuityRecord | append-only |

### 6.2 ResumePoint 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_resume_point | ct_id, step_id | ResumePoint | id 自動採番 |
| get_resume_point | cr_id | ResumePoint | 状態スナップショット含む |

### 6.3 RestorePoint 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_restore_point | ct_id, dh_id | RestorePoint | id 自動採番 |
| get_restore_point | cx_id | RestorePoint | 復元範囲含む |

### 6.4 Checkpoint 操作

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_checkpoint | ct_id, checkpoint_type | Checkpoint | id 自動採番 |
| list_checkpoints | ct_id | Checkpoint[] | 時系列順 |

### 6.5 Query 操作

| 操作 | 入力 | 出力 | 説明 |
|------|------|------|------|
| trace_history | ct_id | (DH[], OS[]) | Design History と Operation の履歴を取得 |
| find_resume_point | flow_id, step_id | ResumePoint | 再開点を検索 |
| find_restore_point | dh_id | RestorePoint | 復元点を検索 |

---

## 7. Examples（最小例）

本節の例は BSL_1 第10章「Running Example」で定義された共通例に基づく。

### 7.1 最小ContinuityRecord

```json
{
  "id": "CT001",
  "flow_ref": "A001",
  "behavior_ref": "Q001",
  "variant_ref": "V001",
  "operation_ref": "OS001",
  "evidence_chain": ["R001", "C001", "O001"],
  "design_history_ref": "DH001",
  "anchor": {
    "type": "behavior",
    "target_id": "S001",
    "ordering_ref": "O001"
  },
  "timestamp": "2025-06-01T12:00:00Z",
  "description": "対象Aの初期継続性レコード"
}
```

### 7.2 ResumePoint

```json
{
  "id": "CR001",
  "continuity_record_id": "CT001",
  "step_id": "S001",
  "state_snapshot": {
    "flow_state": "A001",
    "variant_selection": "VO001"
  },
  "preconditions": [
    "Condition C001 が成立していること"
  ],
  "created_at": "2025-06-01T12:00:00Z"
}
```

### 7.3 RestorePoint

```json
{
  "id": "CX001",
  "continuity_record_id": "CT001",
  "design_history_id": "DH001",
  "reason": "品質異常発生時の判断復元用",
  "restore_scope": "variant",
  "created_at": "2025-06-01T12:00:00Z"
}
```

### 7.4 Checkpoint

```json
{
  "id": "CP001",
  "continuity_record_id": "CT001",
  "checkpoint_type": "milestone",
  "label": "初期設定完了",
  "resume_point_id": "CR001",
  "restore_point_id": "CX001",
  "created_at": "2025-06-01T12:00:00Z"
}
```

---

## 8. ID Scheme（IDスキーム）

Continuity関連のIDは以下の接頭辞を使用する。

| 要素 | 接頭辞 | 例 | 説明 |
|------|--------|-----|------|
| ContinuityRecord | CT | CT001 | 継続性レコード |
| ResumePoint | CR | CR001 | 再開点 |
| RestorePoint | CX | CX001 | 復元点 |
| Checkpoint | CP | CP001 | チェックポイント |

すべて「C」で始まるが、Condition（C）との衝突を避けるため2文字目で識別。

---

## 9. Extension Points（拡張点）

以下の拡張はBSLの範囲外だが、互換性を損なわない範囲で許容される。

| 拡張 | 説明 | 定義場所 |
|------|------|----------|
| Operation Anchor | Anchor.type に operation を追加し、Session を基準とする | Sandboxes |
| 自動チェックポイント | 定期的な Checkpoint 自動生成 | Sandboxes |
| 差分比較 | ContinuityRecord 間の差分検出 | Sandboxes |
| 技能伝承ワークフロー | 組織的な知識継承プロセス | Sandboxes |
| ブランチ管理 | 派生・分岐の管理 | Sandboxes |

---

## 10. Summary（本章のまとめ）

| 項目 | 内容 |
|------|------|
| 対象 | Continuity（継続性） |
| 位置づけ | 連続性保証レイヤ（全レイヤの上位） |
| 3要素 | History / Chain / Anchor |
| 主要構造 | ContinuityRecord / ResumePoint / RestorePoint / Checkpoint |
| 原則 | Append-only / Two-layer History / Separation / Resumability / Restorability / Closure |
| 最小単位 | 個人（利用者単位） |
| IDスキーム | CT / CR / CX / CP |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | - | 初版（思想説明を含む旧版） |
| v0.2 | 2025-06 | Core参照ブロック追加、JSON Schema形式化、IDスキーム整理、思想成分排除 |
| v0.3 | 2026-01 | CP-6（Closure: 比較・評価の成立条件）を追加 |
| v0.3.1 | 2026-03 | 公開前の用語整合パッチを適用 |
