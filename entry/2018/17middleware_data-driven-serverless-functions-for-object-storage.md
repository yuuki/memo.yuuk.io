---
Title: 論文 Middleware'17, Data-Driven Serverless Functions for Object Storage
Category:
- Paper
- Middleware
- "2017"
- Serverless
Date: 2018-01-05T21:30:48+09:00
URL: http://memo.yuuk.io/entry/2018/17middleware_data-driven-serverless-functions-for-object-storage
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/8599973812333806307
CustomPath: 2018/17middleware_data-driven-serverless-functions-for-object-storage
---

- [1]: Josep Sampé, Marc Sánchez-Artigas, Pedro García-López, Gerard París. 2017. Data-Driven Serverless Functions for Object Storage. In Middleware.
- [2]: Josep Sampé. zion. https://github.com/JosepSampe/zion

論文のPDFは http://2017.middleware-conference.org/overview.html から手に入る。

## どんなものか

- オブジェクトストレージ向けのデータドリブンなサーバレスコンピューティングミドルウェアが提案されている。
- データパイプライン上に、処理(computations)を注入することで、同期的なインタラクションを要求するユースケースに適している。例えば、動的コンテンツ生成やインタラクティブなクエリ、personalization、コンテンツの検証、アクセス制御など。

## 先行研究との差分はなにか

- 従来のactive storage ((OpenStack Swiftなど))はデータ局所性を得るために、計算タスクをストレージノードに寄せている。そのため、ストレージノード数の分しかスケールアウトしない、計算リソースを共有するためリソース競合があるという課題がある。
- 従来の非同期なイベントドリブンなサーバレスモデル((AWS Lambdaなど))とは異なり、オブジェクトストアに対するデータの読み書きを同期的にインターセプトできる。これをデータドリブンと呼んでいる。
  - AWS API Gatewayは余分なオーバヘッドともうひとつのAPIが増える
  - Amazon Lambda@EdgeはCloudFrontに対するリクエスト/レスポンスをインターセプトできるが、コンピューティングの制約が厳しい。ヘッダやメタデータの改ざん用にデザインされている。

## 手法の要点はなにか

f:id:y_uuki:20180105192725p:image
[1]Figure 1より引用

- Figure 1のようにstorage nodeとgateway nodeの間にcomputing nodeを配置し、レイテンシを削減する

## 有効性をどのように示しているか

### 実装

f:id:y_uuki:20180105203909p:image
[1] Figure 2より引用

- OpenStack Swift上にserverless frameworkのプロトタイプが実装されている。プロキシ側でリクエストをインターセプトするSwift Middlewareが唯一の変更点。[2]がおそらく公開されている実装。
- computationレイヤは、Figure 2のように、Dockerコンテナで実装されている。

### 評価

- 理想的な条件で、Swiftがstorage nodeでリソース競合することを確認
- DockerとJavaのランタイムの起動時間が、0.85sec - 0.95secでAWS Lambdaの5倍速いことを確認
- 10kBのオブジェクトに対して5,000 GETリクエストしたとき、Zion(提案手法)のオーバヘッドは9ms。ただし、Swiftへの内部ホップが5msなので実質+4ms。
- アクセス制御、イメージリサイズ、署名検証、圧縮のそれぞれのfunctionについて、レスポンスタイムとスケーラビリティを評価し、レスポンスタイムについてはTPSを上げつづけてCPU 100%になるとレスポンスタイムが悪化することを確認し、スケーラビリティについてはstorage nodeを増やしてもTPSが向上せず、computingレイヤのワーカーを増やすとTPSがおおざっぱには線形増加することが示されている

## 議論はあるか

- Amazon Athena、Redshift Spectrum、Facebook Prestoとの関連があり、Prestoのパイプライン実行により不必要なI/Oとレイテンシオーバヘッドを避けるところは、提案手法のモデルに似ているが、これらのシステムは汎用のfunctionを提供するわけではなく、インタラクティブなクエリ実行にフォーカスしている点が異なる。

## 興味深い関連論文はなにか

- Scott Hendrickson, Stephen Sturdevant, Tyler Harter, Venkateshwaran Venkataramani, Andrea C Arpaci-Dusseau, and Remzi H Arpaci-Dusseau. 2016. Serverless computation with OpenLambda. In HotCloud.
- Krste Asanovic and D Patterson. 2014. Firebox: A hardware building block or 2020 warehouse-scale computers. In FAST.
- Peter X Gao, Akshay Narayan, Sagar Karandikar, Joao Carreira, Sangjin Han, Rachit Agarwal, Sylvia Ratnasamy, and Scott Shenker. 2016. Network requirements or resource disaggregation. In OSDI. 249-264.

