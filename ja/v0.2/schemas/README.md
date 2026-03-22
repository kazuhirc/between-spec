schemasは、ログや参照束、診断結果などを機械検査するためのスキーマを収録する。CIゲートはschemasに適合することを前提に、modules/rulesの検査を実行する。スキーマは後方互換で拡張し、破壊的変更はメジャーアップで行う。

- schemasは、event/sidecar/evidence/check reportの最小フィールドを定義し、modules/rulesが要求する入力を保証する。
    
- 拡張は後方互換（field追加、enum追加）で行い、field削除・意味変更はメジャーアップでのみ行う。