---
Title: 論文 TKDE'17, Time Series Management Systems A Survey
Category:
- Paper
- TKDE
- "2017"
- TSDB
- Survey
Date: 2018-01-22T02:44:04+09:00
URL: http://memo.yuuk.io/entry/2018/17tkde_time-series-management-systems-a-survey
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/8599973812339781184
CustomPath: 2018/17tkde_time-series-management-systems-a-survey
---

Cyber Physical Systems((IoTに近い用語))の文脈における時系列データベース(TSDB)のサーベイ論文。論文中では、TSDBをTime Series Management System(TSMS)と表現されている。
Stream Processing((ここでは、書き込み時の丸め処理、サンプリングなどを指す))とApproximate Query Processing(AQP) ((AQPは、サンプリングなどのテクニックで現実的なクエリ実行時間に落とし込む技術の総称という理解 [http://www.vldb.org/conf/2001/tut4.pdf:title]))
TKDEは、SIGMODやVLDBといったトップのデータベース系国際会議と並ぶような位置づけのジャーナルらしい。
[http://memo.yuuk.io/entry/2018/17btw_survey-and-comparison-of-open-source-time-series-databases:title] と比べると、読んでみて相当高品質という印象。
ただし、InfluxDBやPrometheusなど、アカデミアにでていないインダストリアルなTSDBはサーベイ対象ではない。

- [1]: Jensen, Søren Kejser, et al. "Time Series Management Systems: A Survey." IEEE Transactions on Knowledge and Data Engineering, vol.29, no.11, 2017, pp. 2581-2600.

論文のPDFファイルを [https://arxiv.org/abs/1710.01077:title] からダウンロードできる。

## 先行研究との差分はなにか

おそらく明記されていないが、現存するTSMSのオーバビューだけでなく、次世代のTSMSの洞察をあたえることがこのサーベイで目指すこととされている。

## サーベイ手法の要点はなにか

- Google ScholarでTSMSのオーバビューを把握し、関連するカンファレンスや用語、関連リサーチャーを発見する。論文の参考文献、引用文献、全ての会議とジャーナル(SIGMOD、IEEE Big Data、PVLDBなど)、著者の論文(DBLPとGoogle Scholar、プロフィールページの組み合わせ)をデータスースとした。
- 次の13の基準でTSMSを比較している。
  - Architecture, Year, Purpose, Motivatinal Use Case, Distributed, Maturity, Scale Shown, Processing Engine, API, Approximation, Stream Processing, Storage Engine, Storage Layout
- TSMSを次の3つのカテゴリに分類している。
  - internal data stores、external data stores(GorillaやBTrDB、Druidなど)、RDBMS extention
- 各TSMSについて、性能、デプロイメント、機能などについて定性的にまとめている。

## 議論はあるか

- 分散TSMSは既存の外部DBMSを用いて開発されている一方で、内部データストアをもつTSMSは主に非分散システム。内部データストアTSMSは主に組み込みデバイス用かPoCであり、既存の外部DBMSをもつTSMSはビジネスクリティカルなところで使われる。
<!-- - 既存のRDBMSを拡張したTSMSはほとんどない。 -->
- ドメインエキスパートがユーザ定義のメソッドやモデルを使って拡張できるインタフェースをもつものは一般的でない。
- TSMSはリアルタイム更新、ユーザ定義関数によるストリームプロセッシング、historical dataとincoming dataの両方に対するクエリ実行を提供すべきである。

## 興味深い関連論文はなにか

多すぎるので割愛。TSDBに特化した部分であるpre-aggregation、approximate queryの各アーキテクチャの詳細や、AQPやNoSQLの比較などデータベース一般の観点で読みたいものがいろいろでてきた。
