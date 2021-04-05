---
title: "CentOS7.3 環境(Vagrant)に Mattermost を導入してみるの巻"
emoji: "📝"
type: "tech"
topics: centos7 Vagrant Mattermost Windows
published: true
---

# Mattermost ってなんじゃい
いわゆるSlackクローン。GitLab が OmnibusパッケージでMattermostを提供してる物を指します。ただ同様のSlackクローン「Gitter」をつい先日買収したばかりなので、Mattermost とのすみ分けがどうなるの？と思ったですが、どうやら[平行提供](https://www.infoq.com/jp/news/2017/03/gitlab-gitter-acquisition)するらしいですね。※将来的に一本化されるんじゃない？って個人的には思いますけどね。

# Mattermost のインストール
GitLab Omnibus, Docker, VM, 単体 と色々パッケージ化されているので好きなのを使えばよろし。わたしは、Vagrant環境でインストールしてみました。

# 前提条件
https://docs.mattermost.com/guides/administrator.html#installing-mattermost

によると、PostgreSQL or MySQL どちらか使うことになるので、最近MySQL使いなので「MySQL」推しで。その他は、以下の条件で設定しました

| ソフトウェア | バージョン |
|:-----------|:----------|
|Java |8      | 
|MySQL         |5.7 以上    |
|Nginx|     | 

ちなみに　Mattermost 4.0.0  (4.0.1) ※2017/07/31時点

## MySQL の設定
CentOS7.x では、mariaDB が初期セットアップされている場合があるので資産をあらかじめ削除しておく。
https://weblabo.oscasierra.net/installing-mysql57-centos7-yum/

またMattermost では、MySQL 5.7系をご所望であるので下記の手順に従ってインストールする。
https://docs.mattermost.com/install/install-rhel-71.html#installing-mattermost-server

```bash:
$ sudo yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
$ sudo yum install mysql-community-server
```

お約束の自動起動＆MySQL起動。なんだかrootの初期パスワードが ```/var/log/mysqld.log``` に出力されてるので起動したらログを確認しメモしておきます。

```bash:
$ sudo systemctl enable mysqld.service
$ sudo systemctl start mysqld.service
$ sudo grep 'temporary password' /var/log/mysqld.log
2017-07-26T01:40:31.134735Z 1 [Note] A temporary password is generated for root@localhost: ShKzSkkNv3,?
```

rootのパスワードを変更しておきます。

```bash
$ mysql -u root -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password42!';
```


### MySQL をインストールしたあとに行うセットアップ
https://weblabo.oscasierra.net/mysql-57-init-setup/

```text
$ mysql_secure_installation 

Securing the MySQL server deployment.

Enter password for user root: 
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration
of the plugin.
Using existing password for root.

Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : 

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

```/etc/my.cnf``` に下記設定を追加します。

```text
character-set-server = utf8
default_password_lifetime = 0
```

MySQLを再起動します。

```bash
$ sudo systemctl restart mysqld.service
```

### Mattermost データベースとユーザの作成

```bash
$ mysql -u root -p
mysql> CREATE DATABASE mattermost;
mysql> GRANT ALL PRIVILEGES ON mattermost.* TO 'mmuser'@'localhost' IDENTIFIED BY 'mmuser_password' WITH GRANT OPTION;
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
↑ セキュリティポリシーに沿ってないって怒られる

mysql> GRANT ALL PRIVILEGES ON mattermost.* TO 'mmuser'@'localhost' IDENTIFIED BY 'mmuser_Password42!' WITH GRANT OPTION;
```

## Mattermost の設定
最新版をインストールする。 2017/07/26現在 4.0.1 

```bash
$ wget https://releases.mattermost.com/4.0.1/mattermost-4.0.1-linux-amd64.tar.gz
$ tar -xvzf mattermost-4.0.1-linux-amd64.tar.gz
$ sudo mv mattermost /opt
$ sudo mkdir -p /opt/mattermost/data
$ sudo useradd --system --user-group mattermost
$ sudo chown -R mattermost:mattermost /opt/mattermost
$ sudo chmod -R g+w /opt/mattermost
```

接続情報を変更する。※バックアップとっておきましょう！

```bash
$ cd /opt/mattermost/config
$ sudo cp config.json config.json.bk
$ sudo vi config.json
       "DataSource": "mmuser:mostest@tcp(dockerhost:3306)/mattermost_test?charset=utf8mb4,utf8&readTimeout=30s&writeTimeout=30s",
↓
       "DataSource": "mmuser:mmuser_Password42!@tcp(localhost:3306)/mattermost?charset=utf8mb4,utf8",
```

立ち上げてみる

```bash
$ cd /opt/mattermost/bin
$ sudo su mattermost
$ ./platform

DB接続情報が間違っているとき
[2017/07/26 11:54:51 JST] [INFO] Pinging SQL master database
[2017/07/26 11:54:51 JST] [EROR] Failed to ping DB retrying in 10 seconds err=dial tcp: lookup dockerhost: no such host

正しいとき
[2017/07/26 11:57:23 JST] [INFO] Starting Server...
[2017/07/26 11:57:23 JST] [INFO] Server is listening on :8065
[2017/07/26 11:57:23 JST] [INFO] Starting jobs
```

とりあえずこれで localhost:8065 にアクセスすると。。！って確認してみる。
問題なければ、サービスを止めておく（下記のため）。

### Mattermost のサービス化
https://docs.mattermost.com/install/prod-rhel-7.html#id2
をみつつ。Mattermost のサービス化を設定する。

```bash:/usr/lib/systemd/system/mattermost.service
[Unit]
Description=Mattermost
After=syslog.target network.target mysql.service

[Service]
Type=simple
WorkingDirectory=/opt/mattermost/bin
User=mattermost
ExecStart=/opt/mattermost/bin/platform
PIDFile=/var/spool/mattermost/pid/master.pid
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target
```

設定したら。サービスを起動する。

```bash
$ sudo chmod 644 /usr/lib/systemd/system/mattermost.service
$ sudo systemctl enable mattermost
$ sudo systemctl start mattermost
```

## nginxの設定
http://qiita.com/MuuKojima/items/afc0ad8309ba9c5ed5ee

を参照しつつnginxをインストールし。定義を設定します。

```bash:/etc/nginx/conf.d/mattermost.conf
server {
  server_name mattermost.example.com;

  location / {
    client_max_body_size 50M;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Frame-Options SAMEORIGIN;
    proxy_pass http://localhost:8065;
  }
}
```

default.conf は、default.conf.bk とでもして、nginx サービスを再起動します。

これでokですね。

# 参考にさせて頂いたサイト様
> [SlackクローンのMattermostを使ってみる - 導入、初期設定編-](http://qiita.com/terukizm/items/4a4016d8ec5a21856e4f)
> [SlackクローンのMattermostをCentOS+MySQLで立ててみた](http://qiita.com/st_1t/items/e9efebe7aa7be8d57863#hubot)
> [Mattermost のインストール for CentOS 7.2](http://qiita.com/shadowhat/items/c29a6d0362a2742425ba)





