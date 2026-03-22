# BSL Annex B. Variation Policy Schema

**Version: v0.3.3**

---

## Core Dependency

本 Annex が依拠する Core の定義を以下に示す。

| 参照先 | Core節 | 本 Annex での役割 |
|--------|--------|---------------------|
| Variation | A.4.2 | Identity を壊さない範囲で許容される揺れ |
| Variation Policy | A.4.3 | どこまで揺れてよいかの宣言 |
| Meaning Identity | A.4.1 | 同一性判定の基準 |
| Sidecar | A.5.2 | Variation Policy の格納先 |
| Semantic Discontinuity | A.4.4 | 比較不能状態（severity とは別概念） |

BSL 独自の定義：VariationPolicy（機械可読な許容範囲の宣言構造）

---

## 1. Purpose（目的と非目的）

### 1.1 目的

本 Annex は、Variation（許容される揺らぎ）を機械可読にするための最小表現を提供する。

Core 7章では Variation を「Identity を壊さない範囲で許容される揺れ」と定義し、Sidecar に宣言するとしている。
本 Schema はその宣言形式の相互運用を可能にする。

### 1.2 非目的

本 Annex は以下を規定しない。

| 非目的 | 理由 |
|--------|------|
| 統計手法（Cp, Cpk, GR&R 等） | ドメイン固有の品質管理手法は Sandboxes に委ねる |
| ドメイン固有の許容設計 | JIS / ISO / ASME 等の規格適合は Sandboxes に委ねる |
| 式言語の評価方法 | functional 型の expr 実行は実装依存 |
| 単位変換アルゴリズム | 実装層の責務 |

### 1.3 Schema の適用範囲

本 Annex が規定するのはベース構造（id / type / basis / applies_to / unit / severity / status / superseded_by）のみである。

type 固有のフィールド（value / min / max / values / expr 等）は参考定義であり、JSON Schema としての厳密な検証は実装に委ねる。

---

## 2. Data Model（データモデル）

### 2.1 VariationPolicy

許容範囲を宣言する構造。

#### Schema（ベース構造）

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^VP[0-9]{3,}$",
      "description": "Variation Policy ID（例：VP001）"
    },
    "type": {
      "type": "string",
      "enum": ["absolute", "relative", "range", "enum", "functional"],
      "description": "許容範囲の種別"
    },
    "basis": {
      "type": "string",
      "description": "何に対する Variation か（Flow/Behavior/Evidence 要素への参照 ID）"
    },
    "applies_to": {
      "type": "string",
      "description": "適用範囲（View/Sequence/Ordering 等への参照 ID）"
    },
    "unit": {
      "type": "string",
      "description": "単位（必要な型のみ）"
    },
    "severity": {
      "type": "string",
      "enum": ["info", "warning", "fail"],
      "default": "fail",
      "description": "逸脱時の重大度"
    },
    "status": {
      "type": "string",
      "enum": ["active", "superseded"],
      "default": "active",
      "description": "有効状態"
    },
    "superseded_by": {
      "type": "string",
      "pattern": "^VP[0-9]{3,}$",
      "description": "代替する VariationPolicy ID（status=superseded のときのみ使用）"
    }
  },
  "required": ["id", "type", "basis"]
}
```

#### 必須フィールドの適用範囲

| 区分 | フィールド | 必須化の主体 |
|------|-----------|-------------|
| ベース構造 | id, type, basis | BSL（本 Annex） |
| type 固有 | value, min, max, values, expr 等 | 各実装 |

ベース構造の3項（id / type / basis）のみ BSL が必須と規定する。type 固有フィールドの必須化は各実装が行う。

#### Field Definition

| フィールド | 型 | 必須 | 制約 | 説明 |
|-----------|-----|------|------|------|
| id | string | ○ | `^VP[0-9]{3,}$` | プロジェクト内で一意 |
| type | string | ○ | enum | 許容範囲の種別 |
| basis | string | ○ | 要素 ID 参照 | 何に対する Variation か |
| applies_to | string | - | 要素 ID 参照 | 適用範囲 |
| unit | string | - | - | 単位 |
| severity | string | - | enum | 逸脱時の重大度 |
| status | string | - | enum | 有効状態 |
| superseded_by | string | - | VP ID 参照 | 代替する VariationPolicy |

#### Constraints

| ID | 制約 | 根拠 |
|----|------|------|
| VP-C1 | id はプロジェクト内で一意 | 同一性判定の前提 |
| VP-C2 | basis は既存の Flow/Behavior/Evidence 要素 ID | 参照整合性 |
| VP-C3 | type ごとに必須フィールドが異なる（参考定義） | 型安全性 |
| VP-C4 | superseded_by は status="superseded" のときのみ使用 | 状態整合性 |

#### 参照 ID の推奨形式

basis / applies_to の検証は実装責務だが、相互運用のため以下の prefix を推奨する。

| 対象 | 推奨 prefix | 例 |
|------|-------------|-----|
| Flow 要素 | P / A / L | P001, A001, L001 |
| Behavior 要素 | E / S / Q | E001, S001, Q001 |
| Evidence 要素 | R / C / O | R001, C001, O001 |
| Variant | V / VO | V001, VO001 |
| View | VW | VW001 |

### 2.2 basis と applies_to の区別

| フィールド | 意味 | 例 |
|-----------|------|-----|
| basis | 「何の」Variation か（対象要素） | P001（Part）、R001（Reading） |
| applies_to | 「どの文脈で」適用するか（適用範囲） | VW001（View）、Q001（Sequence） |

basis は必須、applies_to は省略可能（省略時はすべての文脈で適用）。

### 2.3 severity の意味

severity は逸脱時の扱いを指定する。

| severity | 意味 | Core との対応 |
|----------|------|--------------|
| info | Evidence として記録するのみ（判定には使わない） | - |
| warning | 注意喚起。再評価を促す | Recoverable failure になりうる |
| fail | 逸脱（強いシグナル） | Identity failure になりうる（Discontinuity とは別） |

注記：

- severity は原則として明示することを推奨し、default には依存しない
- fail は Identity failure の「候補」であり、Semantic Discontinuity とは別概念である
- Semantic Discontinuity の判定は Core 7.3 の判定順序に従う

### 2.4 status と superseded_by

append-only 運用を支援するためのフィールド。

| フィールド | 意味 | 既定値 |
|-----------|------|--------|
| status | 有効状態 | active |
| superseded_by | 代替する VariationPolicy ID | - |

- status は必須ではない（既定値 active）
- 修正は新規 VariationPolicy を追加し、旧版の status を superseded に変更する
- superseded_by は status="superseded" のときのみ使用を推奨

### 2.5 tolerance の埋め込み形（推奨）

Reading 等の上位構造から VariationPolicy を参照する場合、tolerance は次の2形態を許容できる。
本 Annex は相互運用のため、形態B（参照形）を推奨する。

| 形態 | 名称 | 例 | 用途 |
|------|------|-----|------|
| A | InlineTolerance | `{"type":"absolute","value":0.1,"unit":"mm"}` | 簡易・単発の許容範囲 |
| B | PolicyRef | `{"variation_policy_id":"VP006"}` | 再利用・変更履歴（superseded）と整合 |

注記：

- 形態Aは VariationPolicy 本体（id/basis/status 等）を含まない最小形である
- 形態Bは Sidecar に格納された VariationPolicy を参照する

---

## 3. Type Definitions（型定義）

以下は type ごとの参考定義である。JSON Schema としての厳密な検証は実装に委ねる。

### 3.1 absolute（絶対許容差）

公称値からの絶対的な許容範囲。

#### 追加フィールド（参考）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| value | number | ○ | 許容差（±value） |
| nominal | number | - | 公称値（基準点） |

#### 例

```json
{
  "id": "VP001",
  "type": "absolute",
  "basis": "P001",
  "value": 0.1,
  "unit": "mm",
  "nominal": 50.0,
  "severity": "fail",
  "status": "active"
}
```

意味：P001 の公称値 50.0 mm に対し ±0.1 mm の許容差

### 3.2 relative（相対許容差）

公称値に対する比率での許容範囲。

#### 追加フィールド（参考）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| value | number | ○ | 許容比率（0.05 = ±5%） |
| nominal | number | - | 公称値（基準点） |

#### 例

```json
{
  "id": "VP002",
  "type": "relative",
  "basis": "R001",
  "value": 0.05,
  "nominal": 100.0,
  "severity": "warning",
  "status": "active"
}
```

意味：R001 の公称値 100.0 に対し ±5% の許容差

### 3.3 range（範囲指定）

最小値・最大値による範囲指定。

#### 追加フィールド（参考）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| min | number | ○ | 最小値 |
| max | number | ○ | 最大値 |
| inclusive | boolean | - | 端点を含むか（既定: true） |

#### 例

```json
{
  "id": "VP003",
  "type": "range",
  "basis": "R001",
  "applies_to": "Q001",
  "min": 18,
  "max": 25,
  "unit": "Cel",
  "inclusive": true,
  "severity": "fail",
  "status": "active"
}
```

意味：R001（Reading）の温度は Q001（Sequence）実行時に 18〜25℃ の範囲

注記：inclusive の既定値は true。片側開区間が必要な場合は functional 型を使用する。

### 3.4 enum（列挙）

許容される離散値の列挙。

#### 追加フィールド（参考）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| values | array | ○ | 許容値のリスト |

#### 例

```json
{
  "id": "VP004",
  "type": "enum",
  "basis": "M001",
  "values": ["MO001", "MO002", "MO003"],
  "severity": "fail",
  "status": "active"
}
```

意味：M001（Material）の選択肢は MO001, MO002, MO003 のいずれか

### 3.5 functional（関数的条件）

式による条件指定。式言語は規定せず、文字列として保持する。

#### 追加フィールド（参考）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| expr | string | ○ | 条件式（評価器は実装依存） |
| params | object | - | 式で使用するパラメータ |

#### 例

```json
{
  "id": "VP005",
  "type": "functional",
  "basis": "R001",
  "applies_to": "Q001",
  "expr": "abs(value - nominal) / nominal < threshold",
  "params": {
    "nominal": 50.0,
    "threshold": 0.02
  },
  "severity": "warning",
  "status": "active"
}
```

#### 注記

- expr は文字列として保持し、BSL は評価方法を規定しない
- 評価器は Sandboxes またはアプリケーション側に委ねる
- params は expr 内で参照可能な変数を定義する
- 安全性・実行環境は実装の責務

---

## 4. Unit Convention（単位表記規約）

unit フィールドは任意だが、相互運用のため以下を推奨する。

### 4.1 推奨コード体系

UCUM（Unified Code for Units of Measure）を推奨する。

| 量 | 推奨表記 | 説明 |
|----|----------|------|
| 長さ | mm | ミリメートル |
| 角度 | deg | 度 |
| 温度 | Cel | 摂氏（℃ は表示用、データは Cel） |
| 時間 | s | 秒 |
| 比率 | 1 | 無次元（0.05 = 5%） |

### 4.2 注記

- 表示用の単位記号（℃、°など）とデータ用のコード（Cel、deg など）は区別する
- 単位変換は実装の責務であり、BSL は規定しない

---

## 5. Versioning Policy（バージョニング規則）

### 5.1 互換性原則

| 変更種別 | 許容 | 説明 |
|----------|------|------|
| フィールド追加 | ○ | 既存の VariationPolicy は影響を受けない |
| type 追加 | ○ | 新しい許容範囲の種別を追加可能 |
| type の意味変更 | × | 既存の解釈を壊す |
| 必須フィールドの追加 | × | 既存の VariationPolicy が無効になる |
| フィールド削除 | × | 既存の VariationPolicy が無効になる |

### 5.2 append-only の適用

VariationPolicy は Sidecar に格納される。
Sidecar の append-only 原則（Core A.5.2）に従い、以下を遵守する。

- 既存の VariationPolicy は削除・上書きしない
- 修正は新規 VariationPolicy を追加し、旧版の status を superseded にする
- superseded_by フィールドで新版を参照可能

#### superseded の例

```json
{
  "id": "VP001",
  "type": "absolute",
  "basis": "P001",
  "value": 0.1,
  "unit": "mm",
  "status": "superseded",
  "superseded_by": "VP010"
}
```

---

## 6. ID Scheme（IDスキーム）

| 要素 | 接頭辞 | 例 | 説明 |
|------|--------|-----|------|
| VariationPolicy | VP | VP001 | 許容範囲の宣言 |

---

## 7. Examples（使用例）

### 7.1 Sequence-level check での使用

BSL_4_Evidence 6.6.1 および BSL_7_Operation 7.5 との連携例。

```json
{
  "id": "R120",
  "value": {
    "kind": "sequence_check",
    "check_id": "CHK-001",
    "predicate": "duration_within_tolerance",
    "result": "pass",
    "tolerance": {
      "variation_policy_id": "VP006"
    }
  },
  "target_behavior_id": "Q001",
  "ordering_id": "O050"
}
```

### 7.2 非対称許容差

上限・下限が異なる場合。

```json
{
  "id": "VP007",
  "type": "range",
  "basis": "P001",
  "min": 49.95,
  "max": 50.10,
  "unit": "mm",
  "severity": "fail",
  "status": "active"
}
```

意味：P001 は 49.95〜50.10 mm の範囲（非対称）

### 7.3 superseded による修正

```json
[
  {
    "id": "VP001",
    "type": "absolute",
    "basis": "P001",
    "value": 0.1,
    "unit": "mm",
    "severity": "fail",
    "status": "superseded",
    "superseded_by": "VP010"
  },
  {
    "id": "VP010",
    "type": "absolute",
    "basis": "P001",
    "value": 0.05,
    "unit": "mm",
    "severity": "fail",
    "status": "active"
  }
]
```

---

## 8. Summary（本 Annex のまとめ）

| 項目 | 内容 |
|------|------|
| 対象 | Variation Policy（許容範囲の宣言） |
| 位置づけ | BSL 本文から参照される informative supplement |
| ベース構造 | id / type / basis / applies_to / unit / severity / status / superseded_by |
| 型 | absolute / relative / range / enum / functional |
| severity | info / warning / fail（既定: fail、明示推奨） |
| status | active / superseded（既定: active） |
| Core との関係 | Core A.4 を前提とするが逆依存しない |
| 格納先 | Sidecar（append-only） |
| バージョニング | フィールド追加・type 追加は許容、意味変更・削除は禁止 |
| 単位表記 | UCUM 推奨 |

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.3 | 2025-12 | 初版（BSL v0.3 に合わせて作成） |
| v0.3.1 | 2026-01 | BSL Appendix → BSL Annex 命名変更。非規範（informative）であることを明示 |
| v0.3.2 | 2026-03 | Summary の「normative reference」を「informative supplement」に修正。BSL_0 の Informative 宣言との整合 |
| v0.3.3 | 2026-03 | §7.1 の BSL_4 参照先を 6.5.1 → 6.6.1 に修正（BSL_4 v0.6.3 の小節番号修正に追従） |
