# BSL_5. Variant 仕様

**Version: v0.2.2**

---

## Core Dependency

本章が依拠するCoreの定義を以下に示す。

| 参照先 | Core節 | 本章での役割 |
|--------|--------|-------------|
| 外側レイヤ（Variant） | A.6.1 | 選択肢の構造 |
| 外側レイヤと三軸の関係 | A.6.2 | Variant → 三軸（一方向） |
| Meaning Identity / Variation | A.4 | 同一性を壊さない範囲の揺れ |
| Variation Policy | A.4.3 | 許容される揺れの宣言 |
| 依存ポリシー（軸間） | A.3.1 | Evidence / Behavior / Flow 間の許可依存（詳細は Core A.3.1 参照） |

BSL独自の定義：Option / Binding / Rule の三層構造

---

## 1. Purpose（この章の目的）

本章は、Variant（選択肢の構造）を機械可読なデータ構造として仕様化する。

Variantは外側レイヤに属し、三軸（Flow / Behavior / Evidence）を参照するが、三軸から参照されない。

仕様化の範囲：
- Option / Binding / Rule のデータモデル
- 各要素の必須フィールド・制約
- 操作（Create / Evaluate / Select）の定義
- 三軸との依存関係

仕様化の範囲外：
- Variationの意味論的定義 → Core Appendix A.4
- 具体的なバリアント管理の実装例 → Sandboxes

---

## 2. Variant の位置づけ

### 2.1 外側レイヤとしての役割

```
┌─────────────────────────────────────────┐
│            外側レイヤ                    │
│  ┌─────────┐ ┌─────────┐ ┌────────────┐ │
│  │ Context │ │ Variant │ │ Design     │ │
│  │         │ │ ★本章  │ │ History    │ │
│  └────┬────┘ └────┬────┘ └─────┬──────┘ │
│       │          │            │        │
└───────┼──────────┼────────────┼────────┘
        │          │            │
        ▼          ▼            ▼
┌─────────────────────────────────────────┐
│           三軸（参照のみ）               │
│   Flow ←── Behavior ←── Evidence        │
└─────────────────────────────────────────┘
```

### 2.2 Variant と Variation の区別

| 項目 | Variant | Variation |
|------|---------|-----------|
| 定義場所 | 外側レイヤ | Core A.4 |
| 役割 | 選択肢の構造を定義 | 同一性を壊さない揺れを許容 |
| 対象 | 明示的な分岐（A or B） | 暗黙的な許容（±0.1mm） |
| 表現 | Option / Binding / Rule | Variation Policy |

---

## 3. Data Model（データモデル）

### 3.1 Variant（選択肢セット）

選択肢セット全体を表す構造。

#### Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^V[0-9]{3,}$",
      "description": "Variant ID（例：V001）"
    },
    "name": {
      "type": "string",
      "description": "Variant名称"
    },
    "target_axis": {
      "type": "string",
      "enum": ["flow", "behavior", "evidence"],
      "description": "対象となる軸"
    },
    "options": {
      "type": "array",
      "items": { "$ref": "#/$defs/Option" },
      "minItems": 2,
      "description": "選択肢一覧（2件以上）"
    },
    "bindings": {
      "type": "array",
      "items": { "$ref": "#/$defs/Binding" },
      "description": "対象軸要素への紐付け一覧"
    },
    "rules": {
      "type": "array",
      "items": { "$ref": "#/$defs/Rule" },
      "description": "選択ルール一覧"
    },
    "default_option_id": {
      "type": "string",
      "pattern": "^VO[0-9]{3,}$",
      "description": "デフォルトのOption ID"
    }
  },
  "required": ["id", "options"]
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^V[0-9]{3,}$` | プロジェクト内で一意 |
| name | string | - | - | Variant名称 |
| target_axis | string | - | enum | 対象軸（flow/behavior/evidence） |
| options | array | ○ | 2件以上 | 選択肢一覧 |
| bindings | array | - | - | 対象軸要素への紐付け |
| rules | array | - | - | 選択ルール |
| default_option_id | string | - | Option ID参照 | デフォルト選択肢 |

---

### 3.2 Option（選択肢）

選択肢の最小単位。

#### Schema

```json
{
  "$defs": {
    "Option": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^VO[0-9]{3,}$",
          "description": "Option ID（例：VO001）"
        },
        "value": {
          "oneOf": [
            { "type": "string" },
            { "type": "number" },
            { "type": "object" }
          ],
          "description": "選択肢の値"
        },
        "label": {
          "type": "string",
          "description": "表示ラベル"
        },
        "attributes": {
          "type": "object",
          "additionalProperties": true
        }
      },
      "required": ["id", "value"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^VO[0-9]{3,}$` | Variant内で一意 |
| value | any | ○ | string/number/object | 選択肢の値 |
| label | string | - | - | 表示ラベル |
| attributes | object | - | - | 任意の属性辞書 |

#### Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| VO-C1 | id はVariant内で一意 | 選択の明確化 |
| VO-C2 | value は三軸（Flow / Behavior / Evidence）の構造を変更しない | Core A.6.2 |
| VO-C3 | 同一Variant内に最低2件のOption | 選択肢の意味 |

---

### 3.3 Binding（紐付け）

Optionを三軸のどこへ適用するかを定義する。

#### Schema

```json
{
  "$defs": {
    "Binding": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^VB[0-9]{3,}$",
          "description": "Binding ID（例：VB001）"
        },
        "target_id": {
          "type": "string",
          "description": "対象要素のID（例：P001）"
        },
        "target_field": {
          "type": "string",
          "description": "対象フィールド（例：material）"
        },
        "option_ids": {
          "type": "array",
          "items": {
            "type": "string",
            "pattern": "^VO[0-9]{3,}$"
          },
          "description": "適用可能なOption ID一覧"
        }
      },
      "required": ["id", "target_id", "option_ids"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^VB[0-9]{3,}$` | Variant内で一意 |
| target_id | string | ○ | 三軸要素ID参照 | 対象要素のID |
| target_field | string | - | - | 対象フィールド |
| option_ids | array | ○ | Option ID参照 | 適用可能なOption |

#### Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| VB-C1 | target_id は既存の三軸要素を参照 | 参照整合性 |
| VB-C2 | Binding は宣言的であること | 命令的な変更禁止 |
| VB-C3 | 三軸の構造を変更しない | Core A.6.2 |

---

### 3.4 Rule（選択ルール）

どのOptionを使うかを決める条件。

#### Schema

```json
{
  "$defs": {
    "Rule": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "pattern": "^VR[0-9]{3,}$",
          "description": "Rule ID（例：VR001）"
        },
        "condition": {
          "type": "string",
          "description": "条件式（例：temperature > 25）"
        },
        "condition_type": {
          "type": "string",
          "enum": ["expression", "evidence_ref", "manual"],
          "default": "expression",
          "description": "条件の種別"
        },
        "then_option_id": {
          "type": "string",
          "pattern": "^VO[0-9]{3,}$",
          "description": "条件成立時のOption ID"
        },
        "else_option_id": {
          "type": "string",
          "pattern": "^VO[0-9]{3,}$",
          "description": "条件不成立時のOption ID"
        },
        "priority": {
          "type": "integer",
          "minimum": 1,
          "default": 1,
          "description": "優先度（複数Rule時の評価順）"
        }
      },
      "required": ["id", "condition", "then_option_id"]
    }
  }
}
```

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^VR[0-9]{3,}$` | Variant内で一意 |
| condition | string | ○ | - | 条件式 |
| condition_type | string | - | enum | 条件の種別 |
| then_option_id | string | ○ | Option ID参照 | 条件成立時のOption |
| else_option_id | string | - | Option ID参照 | 条件不成立時のOption |
| priority | integer | - | >= 1 | 優先度 |

#### Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| VR-C1 | condition は可搬な形式で記述 | ツール非依存 |
| VR-C2 | Flow に依存しない | 構造変更を前提にしない |
| VR-C3 | Evidence / Behavior を参照可能 | Core A.6.2 |
| VR-C4 | Ruleは評価時に適用、三軸を変更しない | 非破壊性 |

---

## 4. Dependency Policy（依存ポリシー）

### 4.1 外側レイヤと三軸の関係

Variant は三軸を参照するが、三軸から参照されない。

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| Variant | Flow | ○ | Binding の対象として |
| Variant | Behavior | ○ | Rule の条件として |
| Variant | Evidence | ○ | Rule の条件として |
| Flow | Variant | × | Core A.6.2 禁止 |
| Behavior | Variant | × | Core A.6.2 禁止 |
| Evidence | Variant | × | Core A.6.2 禁止 |

### 4.2 外側レイヤ内の関係

| 参照元 | 参照先 | 許可 | 備考 |
|--------|--------|------|------|
| Design History | Variant | ○ | Therefore として採用結果を記録 |
| Design History | Context | ○ | Why / Because の構成要素 |
| Variant | Context | ○ | Rule の条件として |
| Context | Variant | × | 循環防止 |

### 4.3 Sidecar との関係

Variant は Sidecar を持ち得る（選択履歴の蓄積）。

| 項目 | 説明 |
|------|------|
| 選択履歴 | どのOptionがいつ選択されたか |
| 評価ログ | Ruleの評価結果 |
| Design History連携 | 採用理由の記録（BSL_6） |

---

## 5. Operations（操作）

### 5.1 Create

| 操作 | 必須入力 | 出力 | 制約 |
|------|----------|------|------|
| create_variant | options (2件以上) | Variant | id 自動採番 |
| add_option | value | Option | id 自動採番、Variantに追加 |
| add_binding | target_id, option_ids | Binding | id 自動採番、参照整合性チェック |
| add_rule | condition, then_option_id | Rule | id 自動採番 |

### 5.2 Evaluate（評価）

| 操作 | 入力 | 出力 | 説明 |
|------|------|------|------|
| evaluate_rules | Variant, Context | Option ID | Ruleを優先度順に評価し、最初に成立したOptionを返す |
| get_applicable_options | Binding | Option ID[] | Bindingに適用可能なOption一覧を返す |

### 5.3 Select（選択）

| 操作 | 入力 | 出力 | 制約 |
|------|------|------|------|
| select_option | Variant, Option ID | Selection | 三軸を変更しない |
| apply_selection | Selection, Target | Result | Bindingに従って値を適用（読み取り専用） |

---

## 6. Examples（最小例）

本節の例は BSL_1 第10章「Running Example」で定義された共通例に基づく。

### 6.1 最小Variant

対象A の P001 に対する材質選択。

```json
{
  "id": "V001",
  "name": "material_variant",
  "target_axis": "flow",
  "options": [
    { "id": "VO001", "value": "material_A", "label": "標準材" },
    { "id": "VO002", "value": "material_B", "label": "高強度材" }
  ],
  "bindings": [
    {
      "id": "VB001",
      "target_id": "P001",
      "target_field": "material",
      "option_ids": ["VO001", "VO002"]
    }
  ],
  "default_option_id": "VO001"
}
```

### 6.2 Rule付きVariant

Evidenceを参照してOptionを選択する例。

```json
{
  "id": "V002",
  "name": "precision_mode",
  "target_axis": "behavior",
  "options": [
    { "id": "VO003", "value": "standard", "label": "標準モード" },
    { "id": "VO004", "value": "high_precision", "label": "高精度モード" }
  ],
  "rules": [
    {
      "id": "VR001",
      "condition": "R001.value < 50.0",
      "condition_type": "evidence_ref",
      "then_option_id": "VO004",
      "else_option_id": "VO003",
      "priority": 1
    }
  ],
  "default_option_id": "VO003"
}
```

### 6.3 複数Binding（拡張例）

```json
{
  "id": "V003",
  "name": "sensor_variant",
  "options": [
    { "id": "VO005", "value": "sensor_type_A" },
    { "id": "VO006", "value": "sensor_type_B" }
  ],
  "bindings": [
    { "id": "VB002", "target_id": "P001", "target_field": "sensor", "option_ids": ["VO005", "VO006"] },
    { "id": "VB003", "target_id": "S001", "target_field": "mode", "option_ids": ["VO005", "VO006"] }
  ]
}
```

---

## 7. ID Scheme（IDスキーム）

Variant関連のIDは以下の接頭辞を使用する。

| 要素 | 接頭辞 | 例 | 説明 |
|------|--------|-----|------|
| Variant | V | V001 | 選択肢セット |
| Option | VO | VO001 | 選択肢 |
| Binding | VB | VB001 | 紐付け |
| Rule | VR | VR001 | 選択ルール |

三軸のIDスキーム（BSL_1参照）との衝突を避けるため、Variant関連はすべて「V」で始まる。

---

## 8. Extension Points（拡張点）

以下の拡張はBSLの範囲外だが、互換性を損なわない範囲で許容される。

| 拡張 | 説明 | 定義場所 |
|------|------|----------|
| 排他制約 | Option間の排他関係定義 | Sandboxes |
| 階層Variant | Variant内にVariantを含む構造 | Sandboxes |
| Design History連携 | 選択理由の記録 | BSL_6 |
| Context連携 | 前提条件に基づく選択 | BSL_1 |

---

## 9. Summary（本章のまとめ）

| 項目 | 内容 |
|------|------|
| 対象 | Variant（選択肢の構造） |
| 位置づけ | 外側レイヤ |
| 三層構造 | Option / Binding / Rule |
| 依存方向 | Variant → 三軸（一方向） |
| 三軸への影響 | なし（参照のみ、変更禁止） |
| Sidecar | 選択履歴として持ち得る |
| IDスキーム | V / VO / VB / VR |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1 | - | 初版 |
| v0.2 | 2025-06 | Core参照ブロック追加、JSON Schema形式化、IDスキーム整理、思想成分排除 |
| v0.2.1 | 2026-03 | 公開前の用語整合パッチを適用 |
| v0.2.2 | 2026-03 | 公開前整合パッチ：Core Dependency 表の依存ポリシー記述を Core A.3.1 参照に修正 |