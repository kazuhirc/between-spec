# BSL Annex A — Architecture補足

**Version: v0.1.2**

本セクションは BSL_9（Architecture）の補足資料であり、非規範（informative）である。
規範定義は BSL_9 本文および Core Appendix を参照のこと。

---

## B.1 Slot Type Catalog

Slot.type に使用する標準的なBSL概念名の一覧を示す。

### B.1.1 三軸（Flow / Behavior / Evidence）

| type | 対応するBSL概念 | BSL章 | 説明 |
|------|-----------------|-------|------|
| Flow.Part | Part | BSL_2 | 識別可能な形状の最小単位 |
| Flow.Assembly | Assembly | BSL_2 | Part間の親子関係・構成 |
| Flow.Placement | Placement | BSL_2 | Partを空間に配置する基準 |
| Behavior.Event | Event | BSL_3 | 時間軸上の離散的な変化点 |
| Behavior.Step | Step | BSL_3 | Eventの結果として確定した状態 |
| Behavior.Sequence | Sequence | BSL_3 | Stepが連続して並んだ順序構造 |
| Evidence.Reading | Reading | BSL_4 | その瞬間に読み取った値 |
| Evidence.Condition | Condition | BSL_4 | Readingを意味づける前提条件 |
| Evidence.Ordering | Ordering | BSL_4 | 観測がどの順番で行われたか |
| Evidence.ActionTrace | Reading | BSL_4 | 行為の記録（派生Reading）。成功・反映・権限OKを含意しない |
| Evidence.ApprovalRef | Reading | BSL_4 | 承認根拠として提示された参照（approval_id等）。成立判定はContext側 |

### B.1.2 外側レイヤ（Variant / Design History）

| type | 対応するBSL概念 | BSL章 | 説明 |
|------|-----------------|-------|------|
| Variant | Variant | BSL_5 | 選択肢セット |
| Variant.Option | Option | BSL_5 | 選択肢の最小単位 |
| Variant.Binding | Binding | BSL_5 | Optionの紐付け |
| Variant.Rule | Rule | BSL_5 | 選択ルール |
| DesignHistory | DesignHistory | BSL_6 | 判断履歴全体 |
| DesignHistory.Why | Why | BSL_6 | 判断の目的 |
| DesignHistory.Because | Because | BSL_6 | 判断の根拠 |
| DesignHistory.Therefore | Therefore | BSL_6 | 採択内容 |

### B.1.3 実行・連続性（Operation / Continuity）

| type | 対応するBSL概念 | BSL章 | 説明 |
|------|-----------------|-------|------|
| Operation.Session | OperationSession | BSL_7 | 運転セッション |
| Operation.Record | OperationRecord | BSL_7 | 運転記録 |
| Operation.State | OperationState | BSL_7 | 運転状態 |
| Operation.Transition | Transition | BSL_7 | 状態遷移 |
| Continuity.Record | ContinuityRecord | BSL_8 | 継続性レコード |
| Continuity.ResumePoint | ResumePoint | BSL_8 | 再開点 |
| Continuity.RestorePoint | RestorePoint | BSL_8 | 復元点 |
| Continuity.Checkpoint | Checkpoint | BSL_8 | チェックポイント |

### B.1.4 拡張ルール

実装層が独自のtypeを追加する場合、以下のルールに従う。

| ルール | 説明 |
|--------|------|
| 接頭辞 | BSL標準概念と区別するため `Ext.` または実装名を接頭辞とする |
| 対応関係 | 可能な限りBSL標準typeへのマッピングを明示する |
| 文書化 | 独自typeは実装層のドキュメントに定義する |

例：`Ext.CustomReading`, `MyImpl.SpecialPart`

---

## B.2 Layer Schema（参考例）

BSL_9 の補足として示す Layer の JSON Schema 例。

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^LY[0-9]{3,}$",
      "description": "Layer ID（例：LY001）"
    },
    "name": {
      "type": "string",
      "enum": ["os", "implementation", "application"],
      "description": "レイヤ名"
    },
    "description": {
      "type": "string",
      "description": "レイヤの説明"
    },
    "components": {
      "type": "array",
      "items": { "type": "string" },
      "description": "レイヤに含まれるコンポーネント"
    },
    "depends_on": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^LY[0-9]{3,}$"
      },
      "description": "依存するレイヤID（下位レイヤのみ）"
    }
  },
  "required": ["id", "name"]
}
```

### B.2.1 Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| LY-C1 | 下位レイヤは上位レイヤに依存しない | BD-3 |
| LY-C2 | depends_on は自身より下位のレイヤのみ | 依存方向 |
| LY-C3 | name は os / implementation / application のいずれか | 三層モデル |

### B.2.2 標準Layer定義

```json
[
  {
    "id": "LY001",
    "name": "os",
    "description": "BSL - 意味OS層",
    "components": ["Flow", "Behavior", "Evidence", "Variant", "DesignHistory", "Operation", "Continuity"],
    "depends_on": []
  },
  {
    "id": "LY002",
    "name": "implementation",
    "description": "実装層",
    "components": [],
    "depends_on": ["LY001"]
  },
  {
    "id": "LY003",
    "name": "application",
    "description": "アプリケーション層",
    "components": [],
    "depends_on": ["LY002"]
  }
]
```

LY002, LY003 の components は実装ごとに自由に定義する。

---

## B.3 Slot Examples（補足例）

BSL_9 本文の例（Flow.Part）以外のSlot例を示す。

### B.3.1 Evidence.Reading

```json
{
  "id": "slot_def456",
  "type": "Evidence.Reading",
  "payload": {
    "format": "spreadsheet_cell",
    "sheet": "measurements",
    "cell": "B12",
    "value": 49.85,
    "unit": "mm"
  },
  "evidence_chain": [],
  "metadata": {
    "source_file": "inspection_log.xlsx",
    "recorded_at": "2025-06-01T10:30:00Z"
  }
}
```

### B.3.2 Behavior.Step

```json
{
  "id": "slot_ghi789",
  "type": "Behavior.Step",
  "payload": {
    "format": "sequence_item",
    "step_number": 3,
    "description": "初期位置への移動",
    "status": "normal"
  },
  "evidence_chain": ["R001"],
  "metadata": {
    "sequence_id": "Q001"
  }
}
```

### B.3.3 DesignHistory

```json
{
  "id": "slot_jkl012",
  "type": "DesignHistory",
  "payload": {
    "format": "decision_record",
    "why": "精度要件を満たす",
    "because": ["測定値が基準超過"],
    "therefore": ["高精度オプション採用"]
  },
  "evidence_chain": ["R001", "R002"],
  "metadata": {
    "approved_by": "reviewer_A",
    "approved_at": "2025-06-01T14:00:00Z"
  }
}
```

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | — | 初版 |