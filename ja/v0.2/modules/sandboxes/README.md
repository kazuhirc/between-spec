
sandboxesは、摂動・検図・正規化など「揺らして確かめる」ための最小実験環境仕様を定義する。alignmentやfingerprintのように比較前提を安定化する要素はここに収め、$Φ$への束ね方だけを規範として固定する。判定基準そのものは扱わない。


sandboxesに入れるべき最小セット

- sandbox spec（何を揺らすか）：Condition差、写像差（mapping_id）、入力生成の範囲
    
- expected invariants（何を守るか）：Identity Key Setのサンプル
    
- observation protocol（何を記録するか）：Reading.kind、Conditionの束ね方、orderingの起点
    
- stop conditions（比較不能で止める条件）：不足参照のチェック順（U1→U2→U3→U4）

