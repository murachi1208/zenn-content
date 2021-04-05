---
title: "踏み台経由でMySQLにアクセスする方法、多段sshトンネリング"
emoji: "📝"
type: "tech"
topics: sshトンネリング MySQL
published: true
---

# 多段sshトンネリングしてみるｗ
[踏み台経由でMySQLにアクセスする方法、sshトンネリング](http://qiita.com/murachi1208/items/fae39969d1288a9f1b1b) で作った環境で、多段sshトンネリングしてみる。メリットは、php, java で直接MySQLにアクセスできるようになるので（PDOで、sshトンネリングする方法が判らなかっただけですｗ）いいかなと。

![image](https://qiita-image-store.s3.amazonaws.com/0/44540/5440cb8e-bb45-8916-f75e-083df5722343.png)

矢印が間違っていたら指摘してくださいｗ

# windowsのコマンドでssh使うお
コマンドラインで「ssh」が使えればいいので、手っ取り早いのは [msysgit をインスコ](http://d.hatena.ne.jp/xyk/20130920/1379659991)する。open-ssh 入るなうｗ

で、Git Bash を立ち上げます（Cygwinみたいだけど）。ssh コマンドを使えるか確認しておきます。

## 段階的にコマンドを実行する

192.168.11.119:22 <--> localhost:30022 にsshトンネリングする。

```
$ ssh -L 30022:192.168.11.119:22 -fN vagrant@192.168.11.118
vagrant@192.168.11.118's password:
```

192.168.11.121:3306 <--> localhost:30023 にsshトンネリングする。

```
$ ssh -L 30023:192.168.11.121:3306 -fN -p 30022 vagrant@localhost
vagrant@localhost's password:
```

## A5:SQL Mk-2 でアクセスしてみる

![image](https://qiita-image-store.s3.amazonaws.com/0/44540/1c0b8a98-9d93-da1f-9bf3-ff99e87c6eb1.png)

きたー

## PDO 経由でもｗ
phpで当然のことながらｗ

```
$dsn = 'mysql:dbname=mysql;host=localhost;port=30023';
$user = 'root';
$password = '';
 
try{
    $dbh = new PDO($dsn, $user, $password);
}catch (PDOException $e){
    print('Error:'.$e->getMessage());
    die();
}
```

できたー

# 参考にさせて頂きました
> [多段ローカルポートフォワード（PostgreSQL接続）](http://yanor.net/wiki/?SSH%2F%E5%A4%9A%E6%AE%B5%E6%8E%A5%E7%B6%9A%2F%E5%A4%9A%E6%AE%B5%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%9D%E3%83%BC%E3%83%88%E3%83%95%E3%82%A9%E3%83%AF%E3%83%BC%E3%83%89%EF%BC%88PostgreSQL%E6%8E%A5%E7%B6%9A%EF%BC%89)
> [teratermを使用した多段ssh転送Add Star](http://d.hatena.ne.jp/jranar/20120317/1331962461)

