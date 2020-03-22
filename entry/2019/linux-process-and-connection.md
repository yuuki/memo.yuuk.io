---
Title: LinuxでTCP/UDPコネクション状態にプロセス情報を紐付ける方法
Category:
- Linux
Date: 2019-06-08T20:34:41+09:00
URL: https://memo.yuuk.io/entry/2019/linux-process-and-connection
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/17680117127191230320
Draft: true
CustomPath: 2019/linux-process-and-connection
---

Linuxでは、`/proc/net/*`や[Netlinkソケット](https://memo.yuuk.io/entry/2018/06/18/003157)を通じて、TCP/UDPのコネクション情報を取得できる。
しかし、ここで取得したコネクション情報は、コネクションを保有するプロセスに関する情報(pidなど)を含んでいない。
ssコマンドの`--extended`オプションは、inode番号を表示することから、inode番号からプロセス情報を辿ることを考える。

当該プロセスのpidがわかっていれば、`/proc/<pid>/fd`以下から、次のようにsocket inode番号を知ることができる。

```shell
ubuntu@yuukidev01:~$ sudo readlink /proc/21326/fd/6
socket:[16801]
```

しかし、inode番号からそのinodeを保持しているpidを直接知ることはおそらくできない。
そこで、次のような**TCP/UDPのコネクションリスト**と**プロセスリスト**の2つのリストを突き合わせ、socket inodeをキーとして、結合する必要がある。(RDB風に表現するとNested Loop Joinする。)

```
TCP/UDP connections (netlink diag messages | /proc/net/*)
----------------------------------------------------------
| State | Local Address:Port | Peer Address:Port | Inode |
|--------------------------------------------------------|
| ESTAB | 10.0.0.10:80       | 10.0.0.11: 46021  | 16801 |
| ESTAB | 10.0.0.10:80       | 10.0.0.11: 48010  | 16829 |
| ...                                                    |
----------------------------------------------------------
```

```
Processes (/proc/<pid>/*)
------------------------------------------------|
| Pid  | Process Name | File Discriptor | Inode |
| 543  | nginx        |  3              | 16801 |
| 543  | nginx        |  3              | 16802 |
| ...                                           |
-------------------------------------------------
```

iproute2ユーティリティのssコマンドの実装でもこれと同じようになっており、具体的には次の通りである。

1. `--processes`オプションが指定されると、`user_ent_hash_build`が呼ばれる https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L5073 .
1. `user_ent_hash_build`では、`/proc/`以下をスキャンして、[inodeをキーとしたプロセス情報の連想配列](https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L520)を作成している、 https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L573-L674
1. コネクションの各行を表示するときに、コネクションのinodeをキーとして2.の連想配列からプロセス情報を取得する。[tcp_show_line](https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L2607) -> [inet_stats_print](https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L2655) -> [proc_ctx_print](https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L2310) -> [find_entry](https://github.com/shemminger/iproute2/blob/afa588490b7e87c5adfb05d5163074e20b6ff14a/misc/ss.c#L683)

ただし、TCPコネクションのステートがTIME_WAITなど、ソケットをcloseした後のステートでは、ssコマンドでinodeが0となり、プロセス情報を取得できない。
また、まれにステートがESTABであっても、inodeが0となっていることがある。

## 参考

- [https://unix.stackexchange.com/questions/302152/is-there-a-way-to-identify-the-pid-or-cgroup-of-a-socket-without-iterating-thr:title]
- [https://stackoverflow.com/questions/14667215/finding-a-process-id-given-a-socket-and-inode-in-python-3:title]
