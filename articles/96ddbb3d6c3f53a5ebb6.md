---
title: "MariaDB でクラスタ化してみる（MariaDB 10.0 Galera Cluster,CentOS 6.5）を調査した結果のメモメモ"
emoji: "📝"
type: "tech"
topics: mariadb Galera
published: true
---

# MySQLクラスタ化
Oracle RAC などを会社で使っていた（主に使うガワ）のですが、自分でも設定してみたいなと思っていたら[MariaDB Galera Cluster](https://mariadb.com/kb/en/mariadb/yum/)が、比較的設定が簡単だと聞いたのでそのときに調査したメモ。

# 先人達の記事とおりなんだけどね
HA構成ちゃんと考えるなら LVS + Keepalived 使うのがスタンダードっぽいですけど、とりあえずどんな動作をするか確認してみます。ここらへんは、Oracle RACに比べて敷居が低くていいわ。が、業務で使うならバックアップとかリストアとか、クエリーログとるとか、、色々考えないといけませんけど、まあ勉強ですから。

> [MariaDB Galera Clusterを試す](http://research.sakura.ad.jp/2013/02/14/mariadb-galera-cluster-1/comment-page-1/)
> [MariaDB 10.0 Galera ClusterをCentOS7にインストール](http://qiita.com/tukiyo3/items/0971f89b5ad3024f5680#%E6%B3%A8%E6%84%8F%E7%82%B9)

# MariaDB 10.0 Server
MariaDB 5.5 と 10.0 がありますが、先人達の記事とおり 10.0 を入れてみますよ。手順とおり実施するのも勉強ですから。そしてさっくり動きます。

## 注意点がある
・Galera Clusterは最低3台以上での構成にすること。
　　⇒　推奨らしいです。
・クエリーが実行されたDBサーバのみbinlogが記録される。
　　⇒　バイナリログから、リストア・リカバリする場合は注意が必要。
・Primaryキーがないテーブルは、insert のみ使用可能。update, delete は使えない。
　　⇒　検証したら、そのとおりでした。業務では、Primaryキーを作成しないテーブル作るケースがある場面が浮かびませんが…特に問題ないかと（笑）

# チューニングすべき点
MySQLクラスタ化は驚くほど簡単にできました。3台⇒2台を止めて1台にして、2台復旧⇒3台、などしても当然ですがクラスタ化できているわけです。
あとは、HA構成や運用周りですね。こればかりはテンプレがないので考えないといけませんね。

とりあえず、その前にMariaDBの設定

> [MySQL 5.6のインストール後にチューニングすべき項目](http://yakst.com/ja/posts/61)
> [MariaDBの初期設定](http://www.seirios.org/seirios/dokuwiki/doku.php?id=serverapp:database:mariadb:initial-config)
> [keepalivedで仮想IPを体験](http://qiita.com/tukiyo3/items/dc355dd456c0973b5a39)


# クエリーログがでない
```general-log=1``` クラスタ化に伴う仕様なのかでないので、audit pluginでクエリーログを出力する。
よくよく考えるとバイナリーログも出力しているのでそこから取得するということでもいいかもしれないが、この audit つかうと任意のタイミングでon/off切り替えられるのでいいかもしれない。
ただしこのプラグイン使っても、クエリーが実行されたDBサーバのみ記録されるのは当然のことですｗ

> [MariaDBでデータベースを監査して、クエリーを追跡する！](http://qiita.com/hit/items/0c401767822185500cad)





