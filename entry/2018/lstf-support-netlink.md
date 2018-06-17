---
Title: 'TCP接続を集約表示するlstfでNetlinkにより実行速度が1.6倍になった'
Category:
- Linux
- TCP
- Go
Draft: true
---

[https://memo.yuuk.io/entry/2018/03/25/152138:title:bookmark] にて紹介した[lstf](https://github.com/yuuki/lstf)のホスト上のTCPコネクション情報の取得処理において、`/proc/net/tcp`を読みだす代わりに、Netlinkソケットを利用することで、実行速度が1.6倍になった。lstfのバージョン[0.4.0](https://github.com/yuuki/lstf/blob/master/CHANGELOG.md#v040-2018-06-17)で使えるようになる。

## 実験

約40,000接続あるWebサーバ上にて、lstfコマンドの実行時間を名前解決時間を含まずに比較する。
実験環境はEC2のc4.2xlarge、Debian 8.10、Linuxカーネル3.16であり、リバースプロキシとしてnginxが動作している。

接続数は次のコマンドよりだいたい40,000接続であることを確認する。

```
[y_uuki@hoge ~]$ ss -tan | wc -l
39264
```

`/proc/net/tcp`を読む実装では、次のように実行時間は約500msであり、

```
[y_uuki@hoge ~]$ time ./lstf_old -n >/dev/null

real	0m0.532s
user	0m0.432s
sys	0m0.140s
```

Netlinkソケットを利用する実装では、次のように実行時間は約300msとなっており、約1.6倍の速度向上である。10回実行の平均値をとっても、1.68倍の速度向上となった。

```
[y_uuki@hoge ~]$ time ./lstf -n > /dev/null

real	0m0.318s
user	0m0.272s
sys	0m0.052s
```

netlinkを使った同様のパフォーマンス改善については、[https://lwn.net/Articles/650243/:title]が参考になる。
この記事を真似て、perfにより解析したが、有意な結果が得られたなかったため、宿題としたい。

## 実装

Netlinkは、ユーザ空間のプロセスとカーネルとの通信をソケットインタフェースにより提供する。
ssコマンドやipコマンドを含むiproute2パッケージでは、netlinkを利用することで、デバイスを操作し、カーネル内の情報を取得している。((一方で、netstatコマンドやifconfigコマンドを含むnet-toolsパッケージはioctlに基づいている))
Netlinkソケットからソケットの関する情報を取得するには、Socket Monitoring Interfaceを使う。((Socket Monitoring Interfaceは、CRIUをサポートされるために加えられた機能らしい。))

Socket Monitoring Interfaceは、昔はinetファミリーのみのサポートだったが、Linuxカーネル3.3から様々なソケットタイプをサポートするようになった(raw socketなど, [1](https://github.com/torvalds/linux/commit/7f1fb60c4fc9fb29fbb406ac8c4cfb4e59e168d6),[2](https://github.com/torvalds/linux/commit/d366477a52f1df29fa066ffb18e4e6101ee2ad04),[3](https://github.com/torvalds/linux/commit/126fdc3249c9ced2a0d20f916858fec26a445f61))
だいたい新しいものを[sock_diag](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/sock_diag.h)、古いものを[inet_diag](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/inet_diag.h)としているようにみえる。
それに伴い、若干インタフェースが変更されており、netlinkファミリーとして、[TCPDIAG_FAMILY](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/inet_diag.h#L7)を指定していたところを[SOCK_DIAG_BY_FAMILY](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/sock_diag.h#L6)として指定したり、ユーザ空間からカーネルに送信するリクエスト構造体が[inet_diag_req](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/inet_diag.h#L25), [inet_diag_req_v2](https://github.com/torvalds/linux/blob/v4.0/include/uapi/linux/inet_diag.h#L37)となっている。
CentOS5などの古いカーネルでは、inet_diagのみ利用できる。
lstfでは、古いOSをサポートしたいため、inet_diagを利用した。

ユーザ空間のプログラミングモデルは、大雑把には、まずnetlinkソケットを作成し、Socket Monitoringのためのリクエスト構造体を作成し、netlinkメッセージ構造体として、`sendto`/`sendmsg`システムコールでカーネルに送信する。次に`recvmsg`などで受信し、バイト列をパースし必要な情報を得るという流れになる。
Go言語では、netlinkを扱うためのライブラリとして [vishvananda/netlink](https://github.com/vishvananda/netlink)が有名である。
ただし、vishvananda/netlinkはSocket Monitoring Interfaceを提供していないため、ソケット情報を取得しようと思うと、コア部分の生に近いインタフェースでコードを書くことになる。
vishvananda/netlinkのコア部分(`nl`パッケージ)を使いつつ、inet_diagを[ひとしきり実装した](https://github.com/yuuki/lstf/commit/40787db495d83dad44829cba1dbf79e5d4d80fce)が、
最終的には、Goの[elastic/gosigarのlinuxパッケージ](https://github.com/elastic/gosigar/tree/master/sys/linux)を発見したため、これを利用し実装した。

## 参考文献

- RFC3549, Linux Netlink as an IP Services Protocol, 2003, https://tools.ietf.org/html/rfc3549
- Rami Rosen, Linux Kernel Networking: Implementation and Theory, Apress, 2014
- Christian Benvenuti, Understanding Linux Network Internals, O'Reilly Media, 2005
- [http://kristrev.github.io/2013/07/26/passive-monitoring-of-sockets-on-linux:title]
- [https://www.vividcortex.com/blog/2014/09/22/using-netlink-to-optimize-socket-statistics/:title]
- [https://medium.com/@mdlayher/linux-netlink-and-go-part-1-netlink-4781aaeeaca8:title]
- [http://d.hatena.ne.jp/yasui0906/20080304/p1:title]
- [https://github.com/vishvananda/netlink/pull/199:title]
- iproute2 misc/ss.c