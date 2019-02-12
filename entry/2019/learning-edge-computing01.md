---
Title: エッジコンピューティング技術調査メモ
Category:
- Paper
- Survey
- Edge
Draft: true
---

研究所で[ユビキタスデータセンター](https://hb.matsumoto-r.jp/entry/2019/02/08/135354)をやっていくぞということで、エッジコンピューティング技術について調査している。
次のサーベイ論文が、2018年ということもあり、最近のエッジコンピューティングの状況について、よくまとめられている。

- [Bilal Kashif, Khalid Osman, Erbad Aiman, Khan Samee U, "Potentials, trends, and prospects in edge technologies: Fog, cloudlet, mobile edge, and micro data centers.", Computer Networks, vol 130, pp.94-120, 2018](http://sameekhan.org/pub/B_K_2018_CN.pdf)

論文では、170件の文献をもとに、次の図のようにFog computing、Cloudlets、Micro datacenters、Mobile edge computingの4つのエッジコンピューティング技術について、 技術的効果(Potentials)、応用(Applications)、技術的挑戦(Challenges)の3つの観点で整理されている。

[f:id:y_uuki:20190211000603p:image]
(出典: 当該論文のFig. 4. "Edge computing potentials, applications, and challenges")

エッジコンピューティング技術を必要とする背景は、次のようになる。
スマートデバイス、ウェアラブルガジェット、センサーの発展により、スマートシティ、pervasive healthcare、AR、インタラクティブマルチメディア、IoE(Internet of Everything)などのビジョンが現実化しつつあり、レイテンシに敏感で高速応答を必要とする。
従来のクラウドでは、エンドユーザーや端末とデータセンターとの間のレイテンシが大きいという課題がある。
そこで、この課題を解決するために、エッジコンピューティングでは、ネットワークのエッジやエンドユーザーとクラウドデータセンターの間の中間層を設けることにより、レイテンシの最小化とネットワークの中枢の下りと上りのトラフィックの最小化を目指す。

# 技術的効果

## トラフィック負荷の削減

## レイテンシの最小化

## クラウド上の負荷削減

## エンドユーザーデバイスの負荷削減

## エネルギー消費の削減

## データセンターでの計算処理のオフロード

# 分類

## Fog

## Cloudlets

## Micro datacenter

## Mobile edge computing

# アプリケーション

## IoT

## マルチメディア

## エネルギー効率

## スマートリビング

## 医療

## 計算効率

# 技術的挑戦

## リソース管理と確保

## エッジでの汎用コンピューティング

## セキュリティとプライバシー

## スケーラビリティ

## データ抽象

## フォールトトレランスとQoS
