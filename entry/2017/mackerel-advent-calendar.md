---
Title: ウェブシステム内の待ち行列をMackerelで可視化してみる
Category:
- Performance
- MySQL
- Monitoring
- Mackerel
Date: 2017-12-19T22:59:07+09:00
URL: https://memo.yuuk.io/entry/2017/mackerel-advent-calendar
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/8599973812328092879
---

この記事は、[Mackerel Advent Calendar 2017](https://qiita.com/advent-calendar/2017/mackerel)の19日目の記事です。
前日は fullsat_ さんによる [Lambdaを使ってMackerelのアラートをRedmineのチケットにする](https://qiita.com/fullsat_/items/6297bcb436f74c9b0a70) でした。

ウェブシステムの障害発生時に、どのコンポーネントの処理が滞っているかをざっくり知りたいことがあります。
そこで、ウェブシステムは待ち行列の集合体であることに着目し、各コンポーネント状態を把握するダッシュボードを最近作成しています。

待ち行列については、自分もそれほど詳しいわけではありませんが、[https://www.campus.ouj.ac.jp/~maps17/11/11Queueing_pat.pdf:title=待ち行列システム]をみるとざっと把握できます。
簡単な例として、安定した系であれば、リトルの法則により、平均待ち行列数Lは、平均到着率λと平均待ち時間Wの積に等しいことがわかっています。
[http://memo.yuuk.io/entry/2016/03/27/023002:title]

これらのパラメータをウェブシステムに置き換えると以下のような見慣れた要素になります。

- 平均待ち行列数 => ワーカースレッド数など
- 平均到着率 => 単位時間あたりのリクエスト/クエリ数など
- 平均待ち時間 => レスポンスタイムなど

これらのうち2つが判明していれば、残りの1つも推定できます。

## MySQLの場合

[mackerel-plugin-mysql](https://github.com/mackerelio/mackerel-agent-plugins/tree/master/mackerel-plugin-mysql)で取得できるメトリックのうち、平均待ち行列数が`mysql.threads.Threads_Running`、平均到着率が分間クエリ数`mysql.cmd.Com_select` ((簡単のため、SELECTクエリが支配的であるとする))となり、平均待ち時間、つまり平均クエリ実行時間を以下の式グラフで可視化できます。

```
alias(scale(divide(max(role('ServiceA:db-master','custom.mysql.threads.Threads_running')),max(role('ServiceB:db-master','custom.mysql.cmd.Com_select'))), 60000), '推定平均クエリ実行時間 ms')
```

法則というと大仰ですが、実際は時間/クエリ/スレッドを計算しているだけです。
MySQLの場合、クエリ実行時間を取得する一般的な手法がない気がしているため、平均値だけ知るには良い方法となります。
しかし、ネットワークI/O時間を含まないため、ネットワークごしのクライアントからみた実行時間ではないことに注意します。

## Webアプリケーションサーバの場合

Webサーバの待ち行列の状態を知るには、[mackerel-plugin-accesslog](https://github.com/mackerelio/mackerel-agent-plugins/tree/master/mackerel-plugin-accesslog)が便利です。
特に、Webアプリケーションサーバがアクティブワーカー数などのメトリックを直接提供していないケースでは、nginxを同居させてアクセスログを集計しておき、分間リクエスト数と平均レスポンスタイムのグラフから、Webアプリケーションサーバのアクティブワーカー数のグラフを出力できます。


このように、各ロールについて、式グラフと、基となる2つのメトリックのグラフをダッシュボードに並べ、ざっくりどのコンポーネントの調子が悪いかを知る手段として活用しています。
