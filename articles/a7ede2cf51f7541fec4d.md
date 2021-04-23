---
title: "いまさら GitBucket を、3分で動かす環境作り（windowsもokだぜ）"
emoji: "📝"
type: "tech"
topics: GitBucket Windows CentOS
published: true
---

# GitBucket 
ああ判ってますます。いまさらの記事ってｗ
とりあえず GitLab の環境を壊してしまって作るのが手間だったのでさくっと作れる感じのものを探していただけです。そんな訳でこの GitBucket 、ぐぐるとまあ記事沢山でてくるもの判ってます。そんなわけで自分メモ。

# Java8 のインスコ
[Oracle のページ](http://www.oracle.com/technetwork/java/javase/downloads/index.html) からjdkをダウンロードするのだけど、Linuxの場合は rpm を windowsでダウンロードしてから転送して…rpm ってやり方をしていたのだけど便利な方法みつけたｗ

> [Linuxでjdkをwgetする方法](http://qiita.com/hajimeni/items/67d9e9b0d169bf68d1c9)

```
% wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u66-b17/jdk-8u66-linux-x64.rpm"
% sudo rpm -ivh jdk-8u66-linux-x64.rp
```

# GitBucket のダウンロード
https://github.com/takezoe/gitbucket/releases/ から最新版をダウンロードする。
※2016/1/10現在 v3.10.1

# 実行
windowsでもlinuxでも下記のコマンドでok。

```
java -jar gitbucket.war
```

アクセス方法は、localhost:8080 、id/pw は、共に root 。


![image](https://qiita-image-store.s3.amazonaws.com/0/44540/3b8ef6c2-c732-0aff-2793-7c827f63b4f5.png)

はい。ここまで３分でしたね。
ちなみにtomcatとか色々と連携している記事みますが、本家のwikiにはそれ以外の手順が多数説明されています。

> [Welcome to the GitBucket wiki!](https://github.com/gitbucket/gitbucket/wiki)
> [Developer's Guide](https://github.com/gitbucket/gitbucket/blob/master/doc/readme.md)

# どこにデータがあるの？バックアップの方法？そしてバージョーンアップ時は？？
どこにデータがあるの？
Linux であれば ```~/.gitbucket/```
Windows であれば ```%USERPROFILE%\.gitbucket```

バージョンアップ方法
最新の gitbucket.war をダウンロードして差し替えるだけｗ

バックアップの方法

> [GitBucket backup](https://github.com/gitbucket/gitbucket/wiki/Backup)



