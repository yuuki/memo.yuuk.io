---
Title: '読書メモ: 「Designing Data-Intensive Applications」Chapter 1. Reliable, Scalable,
  and Maintainable Applications'
Category:
- DataIntensive
- Books
Date: 2018-04-04T22:50:42+09:00
URL: http://memo.yuuk.io/entry/dataintensive/chapter1
EditURL: https://blog.hatena.ne.jp/y_uuki/performance.hatenablog.jp/atom/entry/17391345971632343959
---

# 感想

Chapter1は、本書のサブタイトルにもなっている、データシステム全体の非機能要件に関する用語(Reliability, Scalability, Maintainability)の定義をしている。
もうわかってるよととばしたくなるけれど、人に説明するときのリファレンス元として重宝する。
Figure 1-1の図のようなことをインフラと呼ぶこともあったけど、インフラが指す範囲が広すぎることとなんでもインフラがやることになってしまい、この層のシステムを指す用語として、データシステム、もしくはData-Intensive Applicationsと名をつけていることがよい。
Scalabilityに関するTwitterのタイムラインアーキテクチャの例が簡潔に紹介されていておもしろかった。

[http://memo.yuuk.io/entry/2017/04/10/002914:title:bookmark]

# メモ

今日のアプリケーションは、<i>compute-intensive</i>に対して<i>data-intensive</i>である。
データベースやキュー、キャッシュを異なる別のカテゴリに所属すると捉えがちだが、「データシステム」という一つの傘の下にいると考えるべきではないか。
最近のデータストレージやデータ処理のためのツールは異なるユースケースをもつ。
そして、多くのアプリケーションには、単一のツールでは満たせない、幅広い要求がある。

[f:id:y_uuki:20180404085516p:plain]

データシステムのデザインには様々なファクターがあるが、ここでは大抵のソフトウェアにとって重要な「Reliability」、「Scalability」、「Maintainability」((SREの考え方では、これらはすべて最終的にはReliabilityにつながるという考え方な気がする))の3つの関心に着目する。

## Reliability

Reliabilityの意味するところは、faultが発生してもシステムが"working correctly"であること。
"fault"は大抵ある一つのコンポーネントの故障を指す。"failure"はシステムが全体としてユーザ提供を停止することを指す。
ハードウェアフォールト、ソフトウェアエラー、ヒューマンエラーがある。
フォールトトーラレンス技術により、エンドユーザからこの種のfaultを隠すことができる。

## Scalability

Scalabilityの意味するとことは、負荷が増大してもパフォーマンスを良好に保つための戦略をもっていること。(「もしシステムが特定の方向に成長するなら、その成長に対処する選択肢は何か？」と「追加の負荷を処理するためのコンピューティングリソースをどのようにして追加するのか？」という問いに答えられる状態)
まず、負荷とパフォーマンスを定量的に表現する方法が必要となる。

Scalabilityのための戦略の例として、例えば、[16]のTwitterのタイムラインの例を考える。

- ツイート投稿: ユーザは新しいメッセージをフォロワーに流せる (4.6k requests/sec on average, over 12k requests/sec at peak).
- ホームタイムライン: ユーザはフォローしている人々が投稿したツイートを閲覧できる (300k requests/sec).

ツイート量だけみるとそれほどではないが、Twitterのスケールには"fan-out"が重要。以下の2つの実装が考えられる。

### アプローチ 1. 

グローバルなツイート集合に新しいツイートを挿入していく。ユーザがホームタイムラインリクエストをすると、自分のフォローのツイートを探索する。

```sql
SELECT tweets.*, users.* FROM tweets
JOIN users ON tweets.sender_id = users.id JOIN follows ON follows.followee_id = users.id WHERE follows.follower_id = current_user
```

[f:id:y_uuki:20180404213018p:plain]

### アプローチ 2.

各ユーザのホームタイムラインのキャッシュをもつ。
[f:id:y_uuki:20180404213023p:plain]

初期のTwitterはアプローチ1.だったが、ホームタイムラインのクエリ負荷に耐えられず、アプローチ2へ移行した。アプローチ2は、書き込み時の負荷は読み込み時の負荷は小さくなる。ツイートの平均投稿レートがホームタイムラインの読み込みレートより2桁小さかったため、うまく動いた。
しかし、平均値では問題なくとも、3000万フォロワーを超える特定のユーザのツイート投稿時に、3000万回の書き込みが発生するという欠点がある。
最終的には、ハイブリッドアプローチをとっており、基本はアプローチ2をとりつつ、特定の巨大フォロワーユーザだけfan-outせず、アプローチ1を採用している。((12章でこの例が再度でてくるとのこと))

## Maintainability

Maintenabilityの本質は、エンジニアリングとオペレーションチームのために生活を良くすること。システムの複雑さを減らし、システムの変更を容易にし、新しいユースケースに対応できる。
以下の3つの設計原則がある。

- Operability: オペレーションチームが、システムをなめらかに動作させ続けやくすること。
- Simplicity: 複雑さを減らして、新しいエンジニアがシステムを理解しやすくすること。
- Evolvability: 将来、エンジニアがシステムを変更しやすくすること。

# 興味深いリファレンス

- [3] Ding Yuan, Yu Luo, Xin Zhuang, et al.: “Simple Testing Can Prevent Most Critical Failures: An Analysis of Production Failures in Distributed Data-Intensive Sys‐ tems,” at 11th USENIX Symposium on Operating Systems Design and Implementation (OSDI), October 2014.
- [11] Richard I. Cook: “How Complex Systems Fail,” Cognitive Technologies Laboratory, April 2000.
- [16] Raffi Krikorian: “Timelines at Scale,” at QCon San Francisco, November 2012.
- [18] Kelly Sommers: “After all that run around, what caused 500ms disk latency even when we replaced physical server?” twitter.com, November 13, 2014.
- [21] Tammy Everts: “The Real Cost of Slow Time vs Downtime,” webperformancetoday.com, November 12, 2014.
- [23] Tyler Treat: “Everything You Know About Latency Is Wrong,” bravenew‐ geek.com, December 12, 2015.
- [24] Jeffrey Dean and Luiz André Barroso: “The Tail at Scale,” Communications of the ACM, volume 56, number 2, pages 74–80, February 2013. doi: 10.1145/2408776.2408794
- [25] Graham Cormode, Vladislav Shkapenyuk, Divesh Srivastava, and Bojian Xu: “Forward Decay: A Practical Time Decay Model for Streaming Systems,” at 25th IEEE International Conference on Data Engineering (ICDE), March 2009.
- [26] Ted Dunning and Otmar Ertl: “Computing Extremely Accurate Quantiles Using t-Digests,” github.com, March 2014.
- [27] Gil Tene: “HdrHistogram,” hdrhistogram.org.
- [28] Baron Schwartz: “Why Percentiles Don’t Work the Way You Think,” vividcortex.com, December 7, 2015.
- [34] Hongyu Pei Breivold, Ivica Crnkovic, and Peter J. Eriksson: “Analyzing Software Evolvability,” at 32nd Annual IEEE International Computer Software and Applications Conference (COMPSAC), July 2008. doi:10.1109/COMPSAC.2008.50

