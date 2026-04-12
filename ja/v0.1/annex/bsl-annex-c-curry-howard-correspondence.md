# BSL Annex C: Curry–Howard Correspondence for Contract and Admissibility (informative)

**Version: v0.2**

## C1 Purpose and status

本 Annex の目的は、Curry–Howard 対応を Between 全体へ拡張することではなく、Contract と admissibility judgment の読解を助ける補助線を与えることにある。ここで主に扱うのは、operation ごとの局所的成立条件束としての Contract と、その Contract の下で当該 operation が admissible かどうかを返す judgment の読みである。したがって本 Annex は informative に留まり、規範定義や運用規則の根拠を新たに与えるものではない。

本 Annex で admissibility と述べる場合、それは Contract の下で当該 operation を進めてよい局所的成立を指す。この語は Core の View と Basis の条件と矛盾しないが、本 Annex ではそれらを operation の成立条件として読むための補助語として用いる。上位概念の所在は Core にあり、規範境界の所在は BSL 本文にあり、operation の require ensure と失敗型の所在は Shell にある。

そのため本 Annex では、Context と Contract の境界を切り、Contract の内側に何が入るかを読み、admissibility judgment を局所判定として位置づけるところまでを主題とする。これに対して、Stop Rule、trace、replayability、handoff、commit、運用上の戻し方や routing は、本 Annex の主戦場ではない。以後の章では、まず Context から Contract がどのように局所化されるかを見たうえで、その Contract を論理対応として読み、最後にその対応がどこまで有効で、どこで止めるべきかを整理する。

## C2 Context and Contract

Context は、ある operation を取り巻く判断前提の全体である。そこには、目的、姿勢、許可範囲、比較の見方、外部依存の扱いなど、当該 operation の外側にも広がる条件が含まれる。これに対して Contract は、その全体をそのまま写したものではない。Contract は Context のうち、当該 operation を特定の Space で成立させるために必要な部分だけを切り出した局所条件束である。

この区別が重要なのは、Context 全体と Contract を同一視すると、operation の成立条件と判断環境の広がりが混ざってしまうからである。Between では、Contract は operation ごとに閉じるべき前提の束として読まれる。したがって Contract は一般的な環境記述ではなく、いまこの操作を admissible と判定するために必要な条件だけを局所的に射影したものである。Context が広い前提空間であるのに対し、Contract はその中で実際に成立判定へ関与する部分に限られる。

ここで先に固定したいのは、Contract が Context 全体の別名ではなく、operation ごとに切り出された局所閉包だという読みである。次章では、その局所閉包の内側に何が入り、どのように admissibility の前提を支えるのかを見る。

## C3 Logical reading of Contract

### Informative correspondence

| Curry–Howard | Between |
|--------------|---------|
| Proposition | Contract |
| Proof | Evidence |
| Program | Operation |

この表は、Contract と admissibility judgment の周辺を読むための最小対応だけを示す informative な参照表である。

この章で Contract を論理対応として読む目的は、Between 全体を単一の一般理論へ還元することではない。ここで見るのは、Contract がある operation を特定の Space で成立させるための前提束として定義され、その読みが admissibility の前提をどのように支えるかという局所的な関係である。Contract は Context 全体の別名ではなく、当該 operation の成立に必要な部分だけを取り出した局所閉包であり、この限定の上でのみ Curry–Howard 的対応は補助線として有効になる。

この局所閉包の内側では、OperationScope は何をどの範囲で行ってよいかを定め、Φ はどの前提の下で評価するかを閉じ、RequiredEvidence は何が揃っていなければ成立と言えないかを固定し、EffectDeclaration は外部依存がある場合にその所在を明示する。したがって Contract は、単に条件を列挙する欄ではなく、operation admissibility のために前提を束ねて閉じる面として読まれる。

ただし、この章で扱う論理対応はここで止めるのがよい。trace、replayability、handoff、commit の詳細や、keep discard retry のような採否語彙は、証跡設計や adoption 側の責務へ広がるため、Contract の論理読解そのものとは分けて扱うほうが境界が保ちやすい。本章では、Contract を admissibility の前提束として読むところまでに留め、judgment の返り方そのものは次章で扱う。

## C4 Admissibility judgment and its boundary with adoption

admissibility judgment は、世界全体についての真偽を返すものではなく、与えられた Contract の下で当該 operation が成立可能かどうかを返す局所判定である。Between 側が所有するのは、何を比較してよいか、どこで止めるか、何を judgment に進めてよいかという admissibility の側であり、trial の前段でその入口条件が閉じていなければ、そもそも judgment へ進んではならない。したがって本章で judgment を読む中心は、一般的な真理値や証明の図式ではなく、局所 Contract の下で operation を通してよいかどうかという成立判定にある。

この意味で judgment は、独立した超越的な判定者ではない。trial が与えた fact を、現時点の basis の下でどう扱うかを返す段であり、その責務は completed trial を保持するか、棄却するか、再試行へ戻すかを append only に残すことである。ここで固定されるのは adoption であって update ではなく、trial 自体が fact を与え、judgment はその fact に対する採否を別に返す。

したがって、admissibility judgment は adoption judgment や update の最終判断そのものとは分けて読まなければならない。keep だけが後段の handoff に進みうるが、その先でも approval や authority が閉じていなければ更新可能性は成立せず、commit はなお別段に残る。次章では、この読みが有効に働く範囲を明示し、どこから先は Stop Rule、trace、handoff、commit など別の責務へ返すべきかを切り分ける。

## C5 Where the correspondence helps and where it should stop

この章で切り分けたいのは、Curry–Howard 的対応がどこまで有効で、どこで止めるべきかである。Contract を operation の局所成立条件束として読み、admissibility judgment をその局所閉包の下での判定として読む限り、この対応は有効な補助線になる。ここまでは、何が揃えば進めるかを論理的に読むことができる。

ただし、その補助線を Between 全体へ広げるのは避けたほうがよい。たとえば keep discard retry は completed trial に対して返される adoption 語彙であり、judgment は completed trial をどう扱うかを append only に残す段であって、世界全体の真偽を閉じる段ではない。また keep は update ではなく、handoff と commit を経て初めて更新可能性と更新そのものが別段に閉じる。したがって、Curry–Howard 的対応が比較的よく効くのは Contract と admissibility judgment の近傍までであり、adoption、update possibility、commit までを一つの証明図式に畳み込まないほうが、既存の責務分離と整合する。

さらに、Stop Rule、trace、replayability、handoff、commit は、局所成立判定そのものというより、停止設計、証跡設計、更新境界の側に属する。したがって本 Annex では、Curry–Howard 的対応を Contract と admissibility judgment の読解補助として限定し、その先の停止語彙、運用分岐、更新手続は Core、BSL 本文、Shell、そして近傍の informative 補助仕様へ返すのが自然である。

## C6 Relation to Core BSL Shell and nearby informative specifications

この章の役割は、Annex C を独立した理論や運用文書として閉じるのではなく、Core、BSL 本文、Shell、および近傍の informative 補助仕様との責務関係の中に置き直すことにある。BSL 本文自身が、Annex を informative 文書とし、規範的主張の根拠としては用いないこと、また Shell を本文とは別の独立仕様として扱うことを明示している。したがって Annex C は、Contract と admissibility judgment の読解を助ける補助線ではあっても、上位概念、規範境界、操作語彙、停止語彙、運用手順の所有者ではない。

上位概念の所在は Core にある。Core は意味の座標系とその読み方の条件を与え、BSL 本文はその座標系を実装が参照できる最小仕様として固定する。これに対して Shell は、require ensure と失敗型を含む語彙と契約を規範として固定する層であり、肥大化しやすい論点や拡張点は本文の外側へ分離する。したがって Annex C の役割は、新しい語彙や新しい運用規則を導入することではなく、すでに置かれている概念と契約の読解を、Contract と admissibility judgment の周辺に限って補助することにある。

そのうえで、trial judgment handoff commit の接続や実行境界の補足は、必要に応じて近傍の informative 補助仕様で展開されうる。ただし、それらは本 Annex の根拠ではなく、ここで切り分けた責務境界を別の面から読むための補助面である。Annex C の自然な位置は、Core の概念、BSL 本文の規範、Shell の操作語彙をつなぐ読解補助に留まり、必要に応じて各論点をそれぞれの所在へ返して読むところにある。

## References

| 参照先 | 関係 |
|--------|------|
| Core DEF-Φ | 評価フレームの定義 |
| Core DEF-Contract | Contract の定義 |
| Core AX-8 | 評価フレームの閉包 |
| BSL Chapter 9 | Space Metadata, Contract, effect_declaration |
| BSL Glossary | Contract, space_id, basis_id, trace_id |
| BSL Shell v0.1 | operation の require ensure と失敗型 |

## 更新履歴

| バージョン | 日付         | 変更内容                                                                                                                                                                      |
| ----- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v0.1  | 2026-01    | 初版。Curry–Howard 対応表、judgement 形式、undefined_type を定義                                                                                                                       |
| v0.2  | 2026-04-12 | Contract と admissibility judgment の読解補助へ再構成。対応表を縮約して C3 に残し、judgement form と undefined_type 表を削除。C4 で adoption との境界を明示し、C6 の参照を Core BSL Shell および近傍の informative 補助仕様に整理 |