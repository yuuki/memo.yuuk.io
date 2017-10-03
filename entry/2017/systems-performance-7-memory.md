---
Title: 詳解システムパフォーマンス 7章「メモリ」メモ
Category:
- System Performance
- Books
Date: 2017-10-04T00:49:58+09:00
URL: http://memo.yuuk.io/entry/2017/systems-performance-7-memory
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/8599973812304475331
CustomPath: 2017/systems-performance-7-memory
---

しばらく輪読を休んでたけど、再開した。第7章「メモリ」。
章末の練習問題を見ながら、みんなで議論していた。

- メモリのページとはなにか
  - OSとCPUがメモリを管理する単位
- 仮想メモリとは何か
  - 無限のメモリの抽象
  - メインメモリとスワップ領域を抽象化している
  - 他に抽象化している組み合わせはあるか？ 分散OSの分野で、分散システム上の各ノードのメモリを一つの仮想メモリとして扱うみたいな研究はあるかも？
- Linuxの用語では、ページングとスワッピングの違いは何か
  - ページングは、メインメモリとスワップデバイスの間のページ転送
  - スワッピングは、スワップファイルかデバイスへのページング
- スワップデバイスのスワップファイルの違い？
  - mkswapでスワップ領域をブロックデバイスかファイルか選べる
  - ファイルのほうがダンプしやすそう。オーバヘッドも大きそう。
- スワップ領域のサイズはいくつに設定する？
  - [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Deployment_Guide/ch-swapspace.html:title]
  - 定期的にメモリをダンプして再開
  - live migration プロセスの状態を保存して移動 コンテナだとこういうの [https://criu.org/Main_Page:title]
- デマンドページングの目的は何か
  - 仮想 => 物理のマッピング作成の遅延評価
  - これがないとCoWができない
- メモリの使用率と飽和を説明しなさい
  - 使用率: ファイルシステムキャッシュは再利用できる
  - 飽和: 図7-6 メモリの解放 で上から順に戦略が使われる
  - リーピングとか実践で使ったことない
- MMUとTLBの目的は何か
  - MMU: 仮想アドレスから物理アドレスへの変換
  - L1だけ仮想アドレス参照なのはなぜか？
    - L2以降はあとでつけたしたから？
    - L1には命令キャッシュが入ってるから？
    - 単に仮想アドレスで参照すれば速いため?
  - TLBは仮想アドレスから物理アドレスへの変換のキャッシュ
  - スレッドあたりのエントリ数が決められている => HyperThreading コンテキストスイッチ発生してもキャッシュが消えないメリットある？
- ページアウトデーモンの役割は何か
  - ページキャッシュの掃除 フリーリストにいれる。kswapd
  - フリーリストはどこ？=> カーネル内。 図7-13
- OOMキラーの役割は何か
  - プロセスを殺してメモリを強制解放する
  - select_bad_process() に選ばれないようにする [https://ccmp.jp/engineerblog/48-oomadj.html:title] [http://wiki.bit-hive.com/north/pg/OOM%A5%AD%A5%E9%A1%BC:title]
- 無名ページングとはなにか。ファイルシステムページングよりもこの種のページングを分析するほうが重要なのはなぜか
  - プロセスのプライベートデータのページング。ヒープ領域やスタック領域など。
  - ファイルシステムキャッシュはOSが追い出せる。プロセスのプライベートデータを解放できるのはアプリケーションのみ。
- フリーメモリが不足したときに、メモリを広げるためにカーネルが取る手段を説明しなさい
  - ページキャッシュ追い出し => スワッピング => リーピング => OOMキラー
  - MyISAMは全部ページキャッシュ バッファプールとかない
  - 最近のLuceneはmmapでインデックスファイルをのせる。昔はJVMヒープ
- スラブベースのアロケーションのパフォーマンス上の利点
  - ページアロケーションのオーバヘッドが小さい
  - ページサイズ以下ならスラブを使う？
  - [http://www.atmarkit.co.jp/flinux/rensai/watch2008/watchmema.html:title]
  - [https://www.ibm.com/developerworks/jp/linux/library/l-linux-slab-allocator/index.html:title]

最後に、[https://speakerdeck.com/yuukit/performance-improvement-of-tsdb-in-mackerel:title:bookmark] でのディスクスラッシング問題について話をした。
激しい random writeにより、ページイン数が爆発し、メモリを飽和させるので、ページアウト数もまた爆発する。ページキャッシュがなくなるため、read時にキャッシュミスし、read I/Oが増加する現象。
posxix_fadvise(2)のPOSIX_FADV_RANDOMにより、write時のページインの先読みを切ることができる。これにより、ページイン数を減らすことができる。
posxix_fadvise(2)のパッチをあてた前後で、`/proc/meminfo`のActive(file)とInactive(file)に変化があった。バッチ前は、Inactiveが多く、パッチ後はActiveが増えた。ActiveとInactiveの定義は以下の通り。

>
Active: Memory that has been used more recently and usually not
        reclaimed unless absolutely necessary.
Inactive: Memory which has been less recently used.  It is more
        eligible to be reclaimed for other purposes
>
https://www.kernel.org/doc/Documentation/filesystems/proc.txt

[asin:4873117909:detail]
