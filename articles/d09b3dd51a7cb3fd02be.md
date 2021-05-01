---
title: "Ubuntu Server 20.04.2 LTS に SQL Server をインストールしてみた"
emoji: "📝"
type: "tech"
topics: ubuntu20.04 SQLServer
published: true
---

# いまさら Ubuntu 触りだしてます
CentOS8のサポート期限問題で最近は、Ubuntu触ったりしてます。ちょうどPi4を触っているので丁度よかったのと、Debian系は割と食わず嫌いだったのでいいかな。と。

・[Ubuntuを入手する](https://jp.ubuntu.com/download)

# ライセンス形態
Microsoft SQL Serverは、利用形態によりライセンスが異なります。Free版でも商用利用できるのでちょっとしたサービスを作るにはいいかもしれない。

・[How to license SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-2019-pricing)
・[SQL Serverは無料で使える？価格プランごとの特徴とは](https://www.fenet.jp/dotnet/column/%E7%94%9F%E6%B4%BB%E3%83%BB%E4%BE%BF%E5%88%A9/245/#SQL_Server-2)

# Microsoft SQL Serverクイック スタート
Microsofのページがわかりやすい。これみればいいかもしれない。手順もシンプルで簡単にインストールできる。SAのパスワードを設定すればよい。

・[クイック スタート:Ubuntu に SQL Server をインストールし、データベースを作成する](https://docs.microsoft.com/ja-jp/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver15)
・[SQL Server on Linux のインストール ガイド](https://docs.microsoft.com/ja-jp/sql/linux/sql-server-linux-setup?view=sql-server-ver15#system)

あとここらへんも

・[UbuntuServer20.04LTSにSQLServer2019をインストールする](https://xn--v6q832hwdkvom.com/post/ubuntuserver20.04lts%E3%81%ABsqlserver2019%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B/)
・[Ubuntu 18.04 LXC に SQL Server をインストールする方法](https://knkomko.hatenablog.com/entry/2020/07/05/025514)

# SSMSでアクセスする
SQL Server Management Studio (SSMS) は、SQL Server のすべてのコンポーネントを構成、管理、開発し、それらのコンポーネントへアクセスするための統合環境で基本的にSQL Serverの設定を行うことができるツール。立ち上げるのにちょっともっさりするけどなれると使いやすい。ただインストール時に毎回やってしまうのが、SSMSダウンロードデフォルトが英語版なんですよね。。。

・[Windows で SQL Server Management Studio を使用して SQL Server on Linux を管理する](https://docs.microsoft.com/ja-jp/sql/linux/sql-server-linux-manage-ssms?view=sql-server-ver15)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44540/d079322a-a9e0-7838-5178-0195bf3fbc4a.png)

# Azure Data Studio
最近は、SSMSでなく Azure Data Studio を使うのがいいらしい。軽くていいかもしれない。VSCodeと同じで日本語化するといいね。

・[Azure Data Studio とは](https://docs.microsoft.com/ja-jp/sql/azure-data-studio/what-is-azure-data-studio?view=sql-server-ver15)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44540/ff9c141e-7c5b-e7b7-f9e1-6025043d2b6f.png)


# Ubuntuの日本語化
日本語関連のパッケージをインストールしてシステムの文字セットを日本語に変更する。

```text
$ sudo apt -y install language-pack-ja-base language-pack-ja ibus-kkc
$ sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"
$ sudo source /etc/default/locale
```
