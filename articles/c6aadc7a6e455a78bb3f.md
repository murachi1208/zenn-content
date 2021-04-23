---
title: "bash-completionで ssh コマンドなどの補完を強化してみる"
emoji: "📝"
type: "tech"
topics: CentOS6.x bash_completion Bash
published: true
---

## Qiitaを眺めていたら
http://qiita.com/kiida/items/3beb1bf718cdc2f0798a さんの記事で bash-completion が紹介されていたので入れてみたｗ

## bash-completion って？
名前から察し…な感じですが bashコマンドの補完をできるようになるパッケージです。つかたとえば ssh でホスト名を指定するとき（/etc/hosts 定義してあるとする）、いちいち /etc/hosts みてホストを…なんて面倒なを ssh + tab ってやると /etc/hosts にあるホスト名がずらずらってでます。※tar コマンドも補完できるぜ

## インストール
yum で入れるだけ（epelレポジトリが必要です）。

```
$ sudo yum -y install bash-completion
```

## 使ってみる
```ssh ```とコマンドを打ってtabを押下とか、``` ls --```とコマンドを打ってtabを押下とか。
まあ便利そうな感じでござる。

## 参考にさせて頂きました
> [bash-completionでserviceコマンドなどの補完を強化しよう](http://heartbeats.jp/hbblog/2013/06/bash-completion.html)


