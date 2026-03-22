
# Between Specification Layer（BSL）

Between 体系は、Core、BSL、Sandboxes の三つのレイヤから構成される。
ここで BSL は Between Specification Layer であり、Between Core と実装側のあいだをつなぐ仕様層である。

BSL は、Flow / Behavior / Evidence によって表現される意味構造を、実装に依存せずに保つために必要な最小限の構造ルールを定義します。
アルゴリズム、ツール、UI、データ形式には踏み込みません。

---

## 目的（Purpose）

BSL が答えようとしている問いは、ひとつだけです。

> 実装の自由を奪わずに、意味の互換性と再現性を保つために、共有されるべき最小構造は何か？

この問いに答えるため、BSL は次を行います。

- Between Core で定義された意味の骨格を固定する
- 構造間の依存関係と境界条件を明確にする
- 実装上の選択（方式・表現・自動化）には介入しない

BSL は実装ガイドではありません。意味指向の仕様層です。

---

## BSL が定義するもの（What BSL Defines）

BSL は以下を仕様として定義します。

- 三軸構造（Flow / Behavior / Evidence）
- 三層モデル（Element / Structure / Basis）
- 依存方向と上書き禁止のルール
- Evidence および Operation の追記専用（append-only）性
- Continuity（継続性）と再開（restart）の意味論

---

## BSL が定義しないもの（What BSL Does NOT Define）

BSL は意図的に次のものを定義しません。

- ファイル形式（CSV、JSON、DWG、SQL など）
- アルゴリズムや自動化ロジック
- ユーザーインターフェースや操作方法
- CAD / PLM / 特定ベンダーへの統合
- 2.5D などの具体的アーキテクチャ

これらはすべて実装サンドボックス、または付録（Annex/Appendix）の責務です。

---

## 読み方（How to Read This Specification）

- まず第0章：位置づけと目的を読む
- 次に第1章〜第4章で三軸構造を把握する
- 用語は Glossary を参照し、表記と定義を固定する
- 以降の章は必要に応じて参照する
- Annex/Appendix は非規範的な補助資料として扱う

---

## Between Core との関係（Relationship to Between Core）

BSL は、Between Core で定義された次の概念を継承します。

- 三軸（Flow / Behavior / Evidence）
- 三層（Element / Structure / Basis）
- 依存ポリシーと境界条件

BSL が行うのは、新しい概念の追加ではなく形式化（formalization）です。

---

## BSL Shell との関係（Relationship to BSL Shell）

BSL Shell は、BSL 本文が定義する意味構造を前提として、
操作的な語彙と契約の最小集合を整理した補助仕様である。
規範定義の正本は Core Appendix および BSL 本文にあり、
Shell はそれらを前提にした操作語彙の整理として読む。

---

## リポジトリ構成（Repository Layout）

このリポジトリは、凍結された規範（v0.1）と、運用資産の追加（v0.2）を分離します。

- v0.2 では次の三点を分離して追加する。
  - playbooks は人間の回復手順
  - rules は機械が返す停止理由（U-type と diagnostic_reason）
  - schemas は両者が共有する入出力の最小構造
- 比較は comparability first で進め、accept と reject の判断は policy-profiles へ分離する。
注記：BSL 本文（第0章〜第9章および Glossary）は `ja/bsl/` に集約し、トップは入口（README）として固定する。
`ja/bsl/` 配下の canonical filename は、version を含めず、小文字ハイフン区切りで管理する。
- BSL 本文の正本は `ja/bsl/` 配下の章別ファイル群である。
- 結合版（通読・配布用）は今後提供予定である。正本は常に `ja/bsl/` 側に置く。
- 修正、差分確認、参照の基準は常に `ja/bsl/` 側に置く。
版情報はファイル名ではなく、本文先頭の Version 表記および更新履歴で管理する。
`v0.1/` など凍結済み文書群は、この命名規則の対象外として version 付き filename を保持してよい。

```
between-spec/
  ja/
    bsl/        # BSL 本文（第0章〜第9章、Glossary）
    ...

  v0.1/         # BSL Shell v0.1（規範、凍結）
    ...         # Shell 仕様本文
    annex/      # 参照資料（非規範）

  v0.2/         # 運用資産（v0.1を変更しない）
    modules/    # 運用に必要な定義
      rules/        # 静的ルール集（CIで落とす）
      sandboxes/    # 摂動・検図の最小実験環境仕様
      playbooks/    # 運用レシピ（手順＋停止点＋回復）
    schemas/    # 機械検査用スキーマ（CIゲート）
    examples/   # 擬似ログ（合否の例、失敗の2段返し例）
    annex/      # ツール固有・判定プロファイル・対話拡張（非規範）
```

v0.1 は意味変更を行いません。v0.2 は v0.1 を変更せず、運用をヒューマン・イン・ザ・ループ（HITL）で回すための資産を追加します。

---

## ライセンス（License）

Between Specification Layer（BSL）は Apache License, Version 2.0 の下で公開されています。

このライセンスは、

- 学術・個人・商用を問わない自由な利用
- 実装の自由度の確保
- 特許による囲い込みの抑止

を同時に満たすために選択されています。

詳細は `LICENSE` および `NOTICE` を参照してください。

---

## 補足（位置づけの整理）

- README
  → 入口・案内・運用上の前提
- 第0章
  → 仕様本文の一部（思想と射程の定義）
- 各章
  → 正式な仕様定義

この分離により、BSL は読みやすさと仕様としての厳密さを両立します。

