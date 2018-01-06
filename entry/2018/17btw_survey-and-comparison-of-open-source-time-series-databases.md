---
Title: 論文 BTW'17, Survey and Comparison of Open Source Time Series Databases
Category:
- Paper
- BTW
- "2017"
- TSDB
- Survey
Date: 2018-01-06T11:23:36+09:00
URL: http://memo.yuuk.io/entry/2018/17btw_survey-and-comparison-of-open-source-time-series-databases
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/8599973812334024739
CustomPath: 2018/17btw_survey-and-comparison-of-open-source-time-series-databases
---

- [1]: B. Mitschang et al. 2017. Survey and Comparison of Open Source Time Series Databases. In BTW. [slide](http://btw2017.informatik.uni-stuttgart.de/slidesandpapers/E4-14-109/slides.pdf)

## どんなものか

- 全てのオープンソース時系列データベース(以下TSDB)の完全なリストと、人気のあるTSDBの機能リストを作成することが目的。
- 機械的に検索した83個のTSDBのうち12個の主要なTSDBを比較する。

## 先行研究との差分はなにか

- 先行研究では、機能比較よりは性能比較にフォーカスしている。
- 先行研究では、すべてのユースケースを満たせない単一のデータベースはないということが示されている。
- この研究では、上記のgapを埋める。((gapが何を指しているかわかりづらかった。))

## 手法の要点はなにか

TSDBを以下の6つの基準グループ(合計27個の基準)で比較している。((この基準は、[NEMAR プロジェクト](http://www.iaas.uni-stuttgart.de/forschung/projects/NEMAR/indexE.php)のコンテクストからきているとのこと [Th15]))

- 1: Distribution/Clusterability
  - 高可用性、スケーラビリティ、ロードバランシング機能で比較
- 2: Functions
  - INS, UPD, READ, SCAN, AVG, SUM, CNT, DEL, MAX, and MIN の各functionで比較
- 3: Tags, Continuous Calculation, Long-term Storage, and Matrix Time Series
- 4: Granularity
  - データ粒度((粒度は解像度とも呼ばれる))、ダウンサンプリング((rollup aggregationと呼ぶこともある。))、最小粒度
- 5: Interfaces and Extensibility
- 6: Support and License

さらに、TSDBを4つのグループに分類している。

- Group 1: TSDBs with a Requirement on other DBMS
- Group 2: TSDBs with no Requirement on any DBMS
- Group 3: RDBMS
- Group 4: Proprietary

## 有効性をどのように示しているか

- 12個のTSDBの選定基準は、Googleの検索ヒット数順。
  - `https://www.google.com/webhp?gws_rd=cr,ssl&pws=0&hl=en&gl=us&filter=0&complete=0`
- 27個の基準を定性的に満たしているかいないかを表で示されている。

f:id:y_uuki:20180106110545p:image
([1]のTab. 3: Comparison of Criteria Group 1: Distribution/Clusterability.より引用)

## 議論はあるか

- Druidが多くの基準を満たすベストな選択肢。他のTSDBと比較して、5つのnode typeをもち、JavaScriptでself-writtenなaggregated functionをサポートする。次点で、InfuxDB, MonetDB, もしくは2つのRDBMS(MySQL, Postgres)。
- 機能でTSDBを差別化できないが、性能で差別化できるかもしれない。
- 次のステップは、repeatableでextensibleで任意のTSDB向けOSSベンチマーキングフレームワーク。

## 興味深い関連論文はなにか

references.

- [Ba16] Bader, A.: Comparison of Time Series Databases, Diploma Thesis, Institute of Parallel and Distributed Systems, University of Stuttgart, 2016. ((学位論文のようなので、質の程度は不明))
- [Wl12] Wlodarczyk, T.: Overview of Time Series Storage and Processing in a Cloud Environment. In: CloudCom. 2012.
- [DMF12] Deri, L.; Mainardi, S.; Fusco, F.: tsdb: A Compressed Database for Time Series. In: Trafic Monitoring and Analysis. Springer, 2012.
- [PFA09] Pungilă, C.; Fortiş, T.-F.; Aritoni, O.: Benchmarking Database Systems for the Requirements of Sensor Readings. IETE Technical Review 26/5, pp. 342Ű349, 2009.
- [Th15] Thomsen, J. et al.: Darstellung des Konzeptes Ű DMA Decentralised Market Agent Ű zur Bewältigung zukünftiger Herausforderungen in Verteilnetzen. In: INFORMATIK 2015. Vol. P-246. LNI, 2015.

citations.

- Jensen, Søren Kejser, et al. “Time Series Management Systems: A Survey.” IEEE Transactions on Knowledge and Data Engineering, vol. 29, no. 11, 2017, pp. 2581–2600.

## see also

- [http://blog.yuuk.io/entry/the-rebuild-of-tsdb-on-cloud:title:bookmark]
- [http://blog.yuuk.io/entry/high-performance-graphite:title:bookmark]
