playbooksは、停止点（checkで止まる場所）と回復手順（最小修正）を短いパイプとして提供する。症状から型エラーへ落とし、必要な参照束と再実行手順を提示することで、属人的なデバッグを減らす。各playbookはv0.1の禁止経路と矛盾してはならない。

playbookの1本は、次で足ります。

- Task Contract（固定）：comparison_scope、$Φ$ミニ（I/A/C/O）、禁止則、採用（approved）の責任境界
    
- Step Contract（差分）：このステップで許可するedge kind、必要Evidence、停止点、回復手順
    
- 擬似ログへのリンク：成功例／失敗例（2段返しが出る例）