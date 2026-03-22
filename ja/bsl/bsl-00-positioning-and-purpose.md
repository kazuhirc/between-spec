# **0. 位置づけと目的（v0.2.4）**

## **0.1 本書の役割：Between と実装の"あいだ"を定義する**

Between は、ものづくりにおける
**空間（Flow）／時間（Behavior）／観測（Evidence）** を統一的に扱うための
「意味の座標系」を与える思想体系である。

一方、実装としての 2.5D Architecture は、
この座標系を用いて **具体的な自動化・配置・帳票・比較・View 操作**を行うための
参照実装である。

本書は、その両者の"あいだ"に置かれる。
すなわち：

- **Between（思想）** … なぜその構造が必要か
- **BSL（仕様）** … 何を最小限として固定するか
- **2.5D（実装）** … どのように実装するか

の三層構造において、
**BSL は「思想を損なわず、実装を縛らない」ための中位レイヤ**として
最小限の骨格だけを形式化する。

Between の意味構造は、Between Core において
Flow / Behavior / Evidence と三層構造（Element / Structure / Basis）として
形式的に定義されている。
本書（BSL）は、この Core の定義を前提に、
実装が参照できる仕様として骨格だけを固定する。

この形式化により、コミュニティによる拡張と、公開・配布・ライセンス等の運用方針を分離したまま両立できる。

### Normative vs Informative（規範と参考の区分）

本リポジトリにおける文書の規範性は以下の通り定義される。

| 区分 | 場所 | 性質 |
|------|------|------|
| Core Appendix | Between Core | Normative（規範） |
| BSL 本文（Chapter 0-9） | BSL | Normative（規範） |
| BSL Annex | BSL | Informative（参考・非規範） |
| Sandboxes | tools/ | Informative（参考・非規範） |

BSL Annex は例・テンプレート・補足説明を提供するが、規範的主張の根拠としては使用しない。
規範定義は Between Core Appendix を参照すること。

---

## **0.2 本書が定義する範囲と定義しない範囲**

本書（BSL: Between Specification Layer）は、Between Core が与える
**意味構造（Flow / Behavior / Evidence と三層構造）** を、
実装が参照可能な **形式仕様（Structure / 梱包単位 / 依存方向 / 記録原則）** に
落とし込むための「意味OS層」である。

BSL が扱うのは、意味の成立に必要な **最小限の構造だけ** であり、
具体的な計算方法・自動化アルゴリズム・UI などには立ち入らない。

### **0.2.1 本書が定義する範囲（BSL の責務）**

BSL の責務は、以下の「揺れてはならない最小骨格」を固定することである。

#### **(1) 三軸（Flow / Behavior / Evidence）と三層（Element / Structure / Basis）の形式仕様**

Between Core で定義された概念をそのまま形式化し、
**境界・整合性・依存方向** を固定する。

三軸の Basis（Placement / Sequence / Ordering）は、
**意味的 SSOT を成立させる基準** として扱う。

| 軸 | Element | Structure | Basis（読み取り基準） |
|----|---------|-----------|------------------|
| Flow | Part | Assembly | Placement |
| Behavior | Event | Step | Sequence |
| Evidence | Reading | Condition | Ordering |

#### **(2) 外側レイヤ（Variant / Design History / Context）の参照構造**

三軸に対して **一方向参照** を定義し、
判断構造が Flow / Behavior / Evidence を上書きしないよう保証する。

意味的同一性（Meaning Identity）と Variation（揺れ）に関する
**判断前提の形式表現** を提供する。

#### **(3) Evidence Chain（append-only）と Operation / Continuity の記録構造**

- Evidence を非破壊的に積み重ねる **append-only の Sidecar 原則**
- Operation による実行ログ
- Continuity による再開点・復元点

を **最小の記録構造** として定義する。
Evidence chain は Sidecar に append-only で蓄積される読み取り履歴であり、View による Meaning Identity の再構成を可能にする。

#### **(4) 依存ポリシー**

- Evidence → Behavior → Flow の一方向依存
- Element → Structure → Basis の層内一方向依存
- 外側レイヤ → 三軸 → Operation → Continuity の上位化
- 上書き禁止・副作用禁止などの **意味保全の原則**

```
許可：Evidence → Behavior → Flow
禁止：Flow → Behavior → Evidence（逆流）

許可：外側レイヤ → 三軸
禁止：三軸 → 外側レイヤ
```

#### **(5) BSL（OS層）と Sandboxes（実装層）との境界条件**

BSL は **意味の座標系だけ** を提供し、
データ形式や操作手続きは実装層に委ねる。

Slot / Layer / Boundary を通じた
**抽象マッピングの枠組み** を提供する。

---

### **0.2.2 本書が定義しない範囲（実装に委ねる部分）**

BSL は具体的な実装方式を規定しない。
以下はすべて **範囲外** とし、Sandboxes や Annex で扱う。

#### **(1) データ形式・ファイル形式**

- Excel / CSV / SQL / Parquet
- AutoCAD DWG / SVG / JSON5 などすべて自由

#### **(2) 実装技術**

- Python / VBA / TypeScript / .NET
- CAD API / メッセージング基盤 / クラウドサービス

#### **(3) UI / ツール / OSS / 自動化アルゴリズム**

- DoubletViewer, CompareAdvanced, Binder
- 自動作図、自動検図、レイアウト生成
- 差分検出・クラスタリング・機械学習

#### **(4) 2.5D Architecture の具象化**

- AutoCAD と Excel を結ぶ具体的な最小アーキテクチャ
- InsertList / ZLEVEL / ZoomPattern などの運用仕様

これらは **Annex（参照実装ガイド）** に退避する。

#### **(5) 公開・配布・ライセンス運用**

BSL は意味構造の仕様であり、公開形態は別文書で扱う。

---

### **0.2.3 BSL の基本方針**

BSL は次の原則に基づいて設計される。

**意味を固定し、表現は自由にする**

意味（構造・依存関係）は共有するが、実装（表現・データ形式）は拘束しない。

**判断は外側レイヤへ、記録は Sidecar へ、構造は三軸へ**

役割を混ぜないことで、揺れの原因となる境界破綻を防ぐ。

**非破壊・再現可能・冪等性の確保**

再計算・再読込・再開が可能な設計を保証する。

---

## **0.3 公開・配布・ライセンスの境界**

本書は、Between の技術仕様（意味構造と境界条件）の規範を定義する。
公開・配布・ライセンス等の運用方針は本書の範囲外とし、別文書で扱う。

BSL 本文が扱うのは、意味の成立に必要な最小限の骨格だけである。
公開形態に依存する主張や、運用上の判断はここでは行わない。

---

## **0.4 想定読者**

本書の読者として想定しているのは、次の層である。

### （1）実装担当者（OSS開発者・社内開発者）

- 2.5D Architecture を書く際の起点が欲しい
- Between の思想を損なわずに実装したい
- Flow・Behavior・Evidence の境界条件が知りたい

### （2）研究者・アーキテクト

- 構造化やモデル化のメタレベルを扱う立場
- MBSE／SysML／オントロジー工学等との接点に関心がある

### （3）OSSコミュニティ

- 拡張・派生を行う際に「揺れてはならない部分」を理解したい
- Between が何を保証し、何を保証しないかの最小セットを知りたい

---

## **0.5 本書の構成**

本書は、Between Core の意味構造を BSL として形式化し、
Sandboxes（実装層）が安全に利用できるように整えたものである。

以下に各章の役割と読み順をまとめる。

### **第1章 Core Concepts（総論）**

三軸（三層）・外側レイヤ・Operation・Continuity の
**全体像・語彙・Core との対応関係** を示す。

- BSL 全域の前提
- 三軸 × 三層の座標系
- Meaning Identity / Variation の定義
- Running Example（第10章）：BSL_2〜BSL_8 の共通例を定義

### **第2〜4章 Flow / Behavior / Evidence（三軸仕様）**

Core の定義に基づき、各軸を

- Element
- Structure
- Basis

の三層に分けて **形式仕様** として固定する。

ここが BSL の三軸を形式仕様として固定する章である。

| 章 | 軸 | Element | Structure | Basis |
|----|----|---------|-----------|-------|
| 2章 | Flow | Part | Assembly | Placement |
| 3章 | Behavior | Event | Step | Sequence |
| 4章 | Evidence | Reading | Condition | Ordering |

### **第5章 Variant（外側レイヤ：選択肢構造）**

- Variant Type（構造選択）
- Variation（揺れ）
- 三軸の Basis との参照関係

など、**判断側の選択肢構造** を形式化する。

### **第6章 Design History（外側レイヤ：判断理由）**

- Why / Because / Therefore の三分割
- Core の「判断レイヤ」の形式化
- Evidence / Variant / Sequence との対応関係

判断過程を append-only で保存し、
将来の比較・検証が可能な履歴構造を与える。

### **第7章 Operation（実行モデル）**

- 実行単位（OperationSession）
- 実行記録（OperationRecord）
- Evidence Chain の形成原則

「何が、いつ、どう実行されたか」を
**非破壊で積み重ねる最小構造** を定義する。

### **第8章 Continuity（継続性・再開点）**

- Snapshot
- Anchor
- Checkpoint / Restore Point
- Time-order Consistency

プロジェクト・設計・実験などの業務を
**途中で止めても再開できる情報構造** を規定する。

### **第9章 Architecture（OS層の境界条件）**

- OS層（BSL）
- Implementation層（2.5D / CAD / DB 等）
- Application層（DoubletViewer等）

三層モデルとして明確に分離し、
**意味OS ⇄ 実装 ⇄ アプリケーション** の境界条件と依存方向を定義する。

Slot / Layer / Boundary を用いて、実装層と BSL を安全に接続する。

補足：
BSL_2〜BSL_8 の例示は、BSL_1 第10章「Running Example」で定義された共通例を前提とする。
本構成表では章番号を再配番せず、参照先のみを明示する。

### **Annex：参照実装ガイド（2.5D Architecture等）**

本文の意味構造を保持したまま、
実装層でのマッピング例・API案・Slot Type Catalog を示す。

- 2.5D Architecture との対応
- DPM（決定論的配置）の実装指針
- View 操作の実装例

---

## **0.6 本書の狙い**

言い換えるなら、本書の役割は次の一言に尽きる。

**"Between の自由さを損なわず、実装の互換性を確保するための最小仕様を与えること。"**

この最小仕様があることで：

- OSSコミュニティは安心して派生実装を作れる
- 実装者はベンダー依存せずに構造化できる
- 2.5D Architecture は抽象的な基盤を得る

本書は、そのための **"揺れない基準"** を提供する。

---

## Core Dependency

本章で述べた BSL の位置づけは、Between Core で規定された形式的構造に基づいている。

### 三軸×三層の継承

BSL は Core が定義した三軸×三層をそのまま継承する。

| 層＼軸 | Flow (`F`) | Behavior (`B`) | Evidence (`E`) |
|--------|-----------|----------------|----------------|
| Element (`El`) | Part | Event | Reading |
| Structure (`St`) | Assembly | Step | Condition |
| Basis (`Ba`) | Placement | Sequence | Ordering |

### 依存ポリシーの継承

BSL は Core が定義した依存ポリシー（A.3）をそのまま継承する。
0.2.1(4) はその要約であり、正式定義は Core Appendix A.3 を参照。

### 境界条件の継承

BSL は Core が定義した境界条件（9章）を継承し、
実装層への非依存性を維持する。

- BSL は意味の成立条件のみを扱い、実装には踏み込まない
- BSL が依存しないのは：データ形式、API、ツール、業務プロセス
- 実装層は BSL の座標系・View/Sidecar原理・Identity/Variation枠組みを継承する

正式定義は Between Core — Appendix を参照のこと。

- 三軸の定義：A.1
- 三層の定義：A.2
- 依存ポリシー：A.3
- 外側レイヤの型：A.6
- 境界条件：9章

---

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-----------|------|----------|
| v0.1    | 2025-11 | 初版 |
| v0.2    | 2025-12 | Core整合強化：0.2を責務・基本方針で再構成、0.5を現行BSL章構成と整合、Core参照セクション追加 |
| v0.2.1  | 2026-01 | Appendix → Annex 命名変更。Normative vs Informative 宣言を追加   |
| v0.2.2  | 2026-03 | 公開前整合パッチ：Basis を「SSOT を成立させる基準」に統一。表ヘッダ・0.5 節の文言を整合 |
| v0.2.3 | 2026-03 | 公開前整合パッチ：§0.5 に Running Example（BSL_1 第10章）への参照を補足 |
| v0.2.4 | 2026-03 | §0.5 第1章の説明に Running Example（第10章）を追記 |

---
