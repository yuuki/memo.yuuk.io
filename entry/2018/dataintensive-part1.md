---
Title: 読書メモ Designing Data-Intensive Applications PART Ⅰ. Foundations of Data Systems
Category:
- DataIntensive
Date: 2018-04-03T19:00:04+09:00
Draft: true
---

「Designing Data-Intensive Applications」がどのような本かは [http://memo.yuuk.io/entry/2017/04/10/002914:title] を参照。 -->

# 章構成

- Chapter1: Reliable, Scalable, and Maintainable Applications
  - 書籍全体を通した用語とアプローチの紹介。reliability, scalability, maintainabilityについて。
- Chapter2: Data Models and Query Languages
  - データモデルとクエリ言語の比較。document, relational, graph modelなどについて。
- Chapter3: Storage and Retrieval
  - ストレージエンジンの内部、ディスク上のレイアウトなど。
- Chapter4: Encoding and Evolution
  - data encoding(serialization)の比較

# Chapter2: Data Models and Query Languages

かつてのデータモデルは1つの巨大なツリー(階層型データモデル)としてデータを表現しようとしていた。
しかし、many-to-manyリレーションをうまく表現できなかったため、リレーショナルデータモデルが発明された。
近年では、開発者はリレーショナルモデルでさえ、いくつかのアプリケーションにフィットしないことを発見した。
リレーショナルモデルでない新しいモデルをNoSQLデータストアと呼び、これらは以下の2つに派生した。

- ドキュメントデータベース: データが自己完結型の"ドキュメント"に入っていて、ドキュメント間の関係はほとんどないユースケース
  - MongoDB, RethinkDB, CouchDB, Espresso
- グラフデータベース: ドキュメントデータベースとは逆に、潜在的にすべてに関係するものがあるユースケース

## ドキュメントデータベース

schema-on-read

```go
if (user && user.name && !user.first_name) {
// Documents written before Dec 8, 2013 don't have first_name
    user.first_name = user.name.split(" ")[0];
}
```

schema-on-write

```sql
ALTER TABLE users ADD COLUMN first_name text;
UPDATE users SET first_name = split_part(name, ' ', 1); -- PostgreSQL
UPDATE users SET first_name = substring_index(name, ' ', 1); -- MySQL
```

ドキュメントデータベースとグラフデータベースは、共通してデータのスキーマを強制しない。
しかし、実際にはデータは確かな構造を持つ。
スキーマがexplicit(schema-on-write)かimplicit(schema-on-read)かという問い。
プログラミング言語の静的型付けor動的型付けの議論に似ている。

次章では、この章のデータモデルを「実装」する上でのトレードオフについて議論する。

## 興味深い文献

- [4] Eric Evans: “NoSQL: What’s in a Name?,” blog.sym-link.com, October 30, 2009. 
- [5] James Phillips: “Surprises in Our NoSQL Adoption Survey,” blog.couchbase.com, February 8, 2012.
- [12] Lin Qiao, Kapil Surlaker, Shirshanka Das, et al.: “On Brewing Fresh Espresso: LinkedIn’s Distributed Data Serving Platform,” at ACM International Conference on Management of Data (SIGMOD), June 2013.
- [15] Sarah Mei: “Why You Should Never Use MongoDB,” sarahmei.com, November 11, 2013.
- [18] Joseph M. Hellerstein, Michael Stonebraker, and James Hamilton: “Architecture of a Database System,” Foundations and Trends in Databases, volume 1, number 2, pages 141–259, November 2007. doi:10.1561/1900000002
- [19] Sandeep Parikh and Kelly Stirman: “Schema Design for Time Series Data in MongoDB,” blog.mongodb.org, October 30, 2013.
- [20] Martin Fowler: “Schemaless Data Structures,” martinfowler.com, January 7, 2013.
- [21] Amr Awadallah: “Schema-on-Read vs. Schema-on-Write,” at Berkeley EECS RAD Lab Retreat, Santa Cruz, CA, May 2009. 
- [22] Martin Odersky: “The Trouble with Types,” at Strange Loop, September 2013.
- [29] Fay Chang, Jeffrey Dean, Sanjay Ghemawat, et al.: “Bigtable: A Distributed Stor‐ age System for Structured Data,” at 7th USENIX Symposium on Operating System Design and Implementation (OSDI), November 2006.
- [31] Herb Sutter: “The Free Lunch Is Over: A Fundamental Turn Toward Concurrency in Software,” Dr. Dobb’s Journal, volume 30, number 3, pages 202-210, March 2005.
- [35] Nathan Bronson, Zach Amsden, George Cabrera, et al.: “TAO: Facebook’s Dis‐ tributed Data Store for the Social Graph,” at USENIX Annual Technical Conference (USENIX ATC), June 2013.


# Chapter3: Storage and Retrieval

ストレージエンジンには、OLTP(Online Transaction Processing)とOLAP(Online Analytic Processing)の2つのカテゴリがある。

OLTP

- 大抵user-facingで大量のリクエストがある
- 各クエリでは少数のレコードにアクセスする
- インデックスを使って、キーに対する値を探索する
- ディスクシークタイムが大抵ボトルネックになる

OLAP

- データウェアハウス、BIなど
- OLTPに比べれば、クエリ数はかなり少ないが、各クエリは大抵要求が厳しく、短時間で数百万のレコードをスキャンする
- ディスクの帯域幅が大抵ボトルネックになる
- インデックスよりは、データをコンパクトにエンコードし、ディスクから読み出すデータ量を最小化することが重要
- カラム指向ストレージ

## 世界で最も単純なデータベース

writeはファイル追記、readは全行scan。

```shell
#!/bin/bash
db_set () {
    echo "$1,$2" >> database
}
db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

```shell
$ db_set 123456 '{"name":"London","attractions":["Big Ben","London Eye"]}' 
$ db_set 42 '{"name":"San Francisco","attractions":["Golden Gate Bridge"]}'
$ db_get 42
{"name":"San Francisco","attractions":["Golden Gate Bridge"]}
```

ストレージエンジンでは以下の2つが重要。

- ログ: 追記のみのデータファイル (append-only)
- インデックス: ある特定のキーに対応する値を効率的に見つけるための、追加のデータ構造
  - readクエリを高速化するが、全てのインデックスは、writeを遅くする

OLTPのストレージエンジンには、log-structuredとupdate-in-place(page-oriented)の2つの主要なインデックスがある。

## Log-structured

キーアイデアは、「ディスク上のランダムアクセス書き込みをシーケンシャル書き込みに変換する」こと。
Bitcask, SSTables, LSM-trees, LevelDB, Cassandra, HBase, Lucene。

### Hash Indexes

インメモリのハッシュマップとKey-Valueペアが追記されたログファイル。RiakのBitcaskなどで利用されている。各キーに対する値が頻繁に更新される状況に向いている。

[f:id:y_uuki:20180402215610p:plain]

追記ログファイルのディスクスペースを削減するにはどうするか？

A. ログが特定サイズに達したとき、ログをセグメントに分割し、新しいセグメントファイルとして切り出す。
セグメント内の重複キーについて、最新の更新だけ残す(compaction)。

[f:id:y_uuki:20180402215708p:plain]

compactionしつつ、セグメントをマージする。

[f:id:y_uuki:20180402215713p:plain]

以下の5つは、現実の実装での工夫。

- ファイルフォーマット: バイナリフォーマットを使うのが速くてシンプル。
- レコードの削除: 削除ログを追記し、compactionのマージプロセス時に削除する
- クラッシュリカバリ: インメモリハッシュマップはロストする。セグメントファイルを全部読めばリカバリは可能だが時間がかかる。そのため、Bitcaskは各セグメントのハッシュマップをディスク上にスナップショットをもつ。
- 中途半端なレコード書き込み: Bitcaskファイルはチェックサムを含む。
- 並行制御: 単一の書き込みスレッドがログの追記を担当する。複数スレッドにより並行読み出しが可能。

追記型のメリットは以下。

- シーケンシャル書き込み。特に磁気ディスクに有効。((CassandraがHDDでも速いという話))
- 並行制御とクラッシュリカバリがよりシンプルになる。valueが上書きされるケースのクラッシュを心配しなくてよい。
- フラグメンテーションを避けられる。

しかし、ハッシュテーブルインデックスは制限もある。

- hash tableはメモリに収まらなければならない。ハッシュマップをディスク上に構築しようとすると、多くのランダムアクセスI/Oが発生する。
- レンジクエリが効率的でない。全てのキーを簡単にスキャンできない。

以下のSSTables(Sorted String Table)などのインデックス構造には、これらの制限がない。

### SSTables and LSM-Trees

セグメントファイル内のキーバリューペアを`sorted by key`な状態に保つ。

1. ファイルが利用可能メモリより大きい状態でも、セグメントのマージ処理がシンプルかつ効率化する。マージソートに近いアプローチ。Figure 3-4。
1. メモリ上のすべてのキーのインデックスをもつ必要がない。スパース構造にすることで、キーのスキャンを速くできる。Figure 3-5。
1. ディスクに書き出す前にレコード群をブロック化し、圧縮し、ディスク・スペースの節約とI/O帯域幅を削減できる。

[f:id:y_uuki:20180402215723p:plain]

[f:id:y_uuki:20180402215725p:plain]

ソートされたデータ構造の構築は、メモリ上であれば赤黒木やAVL木を利用する。ディスク上であればB-tree。
キーの挿入順は不定でも、読み出しはソートされた状態になる。

- インメモリツリーをmemtableと呼ぶ
- memtableが大きくなれば(大抵数MB)、SSTableとしてファイルに書き出す
- read時は、まずmemtableを探索し、直近のディスク上のセグメントファイルを読み、次に古いセグメントを読み...
- バッググランドでマージとcompaction処理が実行され、セグメントファイルが結合し、上書き前の値、削除された値が捨てられる。

このスキーマには一点問題がある。それはクラッシュ時にmemtableの直近の内容が失われること。
ディスク上に別のログを追記で書き込むようにして、クラッシュ前の最後にSSTableにフラッシュした以降のログを読み込みmemtableを修復する。

このインデックス構造をLSM-Tree(Log-Structured Merge-Tree)と呼ぶ。

## Update-in-place (page-oriented)

SSTableと同様に、B-treeが保持するキーバリューペアはキーによりソートされている。
log-structuredインデックスは可変長セグメント、一方、B-Treeは固定長ブロック(またはページ)であり、ディスクハードウェアの構造に近い。

[f:id:y_uuki:20180402215731p:plain]
[f:id:y_uuki:20180402215737p:plain]

B-Treeの書き込み操作は、基本的にディスク上のページの上書きとなり、ページのロケーションを変更しない。
ページの分割書き込みと親ページのリファレンスの更新のように、DBクラッシュ時に親なしページができてしまう可能性がある。
DBをクラッシュセーフにするための一般的な追加データ構造がWAL(write-ahead log, redo log)。WALはB-treeの変更をすべて記録する追記ログファイル。

## LSM-Trees(log-structured)とB-trees(update-in-place)の比較

LSM-treeのアドバンテージ

- LSM-treeは、B-Treeより高い書き込みスループットを維持できる。一回の書き込み操作により発生する追加書き込み(write amprification)とが少ないかつSSTableへのシーケンシャル書き込みのため。HDDだとより顕著になる。
- LSM-treeのほうがより良く圧縮できて、ディスク上のファイル数を小さくできる。B-treeだとページ分割時に空きスペースができるが、LSM-treeは定期的にSSTableをリライトするため。

LSM-treeの欠点

- compaction時のレスポンスタイムの悪化(平均よりは高パーセンタイルへの影響)。B-treeのほうが性能予測しやすい。
- compactionは高い書き込みスループットを発生させる。ディスクの書き込み帯域幅を使い切ってしまう。

## データウェアハウジング

[f:id:y_uuki:20180402215750p:plain]
[f:id:y_uuki:20180402215752p:plain]
[f:id:y_uuki:20180402215758p:plain]
[f:id:y_uuki:20180402215803p:plain]
[f:id:y_uuki:20180402215804p:plain]

# 感想

ストレージエンジンの全体像を確認できた。例えば、OLTP、OLAP、log-structured index、B-treesなど個々の用語を知ってはいても全体としての位置づけ、歴史について知らなかった。

echoとgrepだけの最も単純なデータベースから始まり、compactionについても素朴な実装をベースに理解できた。

# 次に読むべき論文