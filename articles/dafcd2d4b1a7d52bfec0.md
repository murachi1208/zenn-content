---
title: "Vagrant で CentOS6.6, LVS (IPVS) の NAT 転送方法をためして"
emoji: "📝"
type: "tech"
topics: LVS
published: true
---

# LVS (Linux Virtual Server) 
ちょっと勉強する時間ができたので、ロードバランサの勉強をしてみた。[LVS](http://ja.wikipedia.org/wiki/Linux_Virtual_Server) は。。。というか wiki の説明が詳しいので説明はそちらを参考、なのですがvagrant で NAT転送した場合にすこしハマッタとこがあるので毎度の自分メモ。今回、vagrant の nic についてちょっとした小ネタ発見したので結構よかったw

# varant の 複数nic指定
public_network, private_network を同時に指定できるって知らんかった。ただし順番あるのでそこは注意すること。

```
config.vm.define :nictest do |node|
  node.vm.network :public_network, ip: "192.168.0.201" 
  node.vm.network :private_network, ip: "192.168.11.117" 
end
```

と指定した場合、public_network が「eth1」、private_network　が「eth2」でnicが作成される。これ便利やｗ

```
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:4f:b8:06 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
    inet6 fe80::a00:27ff:fe4f:b806/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e0:ee:d2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.201/24 brd 192.168.0.255 scope global eth1
    inet6 fe80::a00:27ff:fee0:eed2/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:10:b4:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.117/24 brd 192.168.11.255 scope global eth2
    inet6 fe80::a00:27ff:fe10:b4f6/64 scope link 
       valid_lft forever preferred_lft forever
```

# ネットワーク構成
図のようなネットワーク構成を作成してみます。①にアクセスするのは、同一ネットワークないの他の機器を想定してます。

![lvs.png](https://qiita-image-store.s3.amazonaws.com/0/44540/293fd70b-e9b6-0ad9-cbaf-0e0c791ca905.png)

```
# lvs
  config.vm.define :nictest do |node|
    node.vm.hostname = "lvs"
    node.vm.network :public_network, ip: "192.168.0.201" 
    node.vm.network :private_network, ip: "192.168.11.117" 
  end

  # node1 
  config.vm.define :node1 do |node|
    node.vm.hostname = "node1"
    node.vm.network :private_network, ip: "192.168.11.118" 
  end

  # node2
  config.vm.define :node2 do |node|
    node.vm.hostname = "node2"
    node.vm.network :private_network, ip: "192.168.11.119" 
  end
```

# lvs（ロードバランサー）側の設定
ipvsadm のインストールと、ポート転送の設定をします。

```
$ sudo yum install ipvsadm
$ sudo vi /etc/sysctl.conf
net.ipv4.ip_forward = 1  ← 0を1に
$ sudo /sbin/sysctl -p
$ sudo /sbin/sysctl -a | grep net.ipv4.ip_forward
```

ipvsadm の設定。DNSラウンドロビン＆NATの設定をします。

```
$ sudo ipvsadm -C
$ sudo ipvsadm -A -t 192.168.0.201:80 -s rr
$ sudo ipvsadm -a -t 192.168.0.201:80 -r 192.168.11.118:80 -m
$ sudo ipvsadm -a -t 192.168.0.201:80 -r 192.168.11.119:80 -m
$ sudo /etc/init.d/ipvsadm save
$ sudo cat /etc/sysconfig/ipvsadm
$ sudo chkconfig ipvsadm on
$ sudo ipvsadm -l
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.201:http rr
  -> 192.168.11.118:http          Masq    1      0          0         
  -> 192.168.11.119:http          Masq    1      0          0         
```

# node1, node2 側の設定
Apache, nginx, node.js なんでもいいですｗ
が、まあ無難に Apache をインストールしておきますかね。

```
$ sudo su
# yum install httpd
# sudo service httpd start
# chkconfig httpd on
# echo "node" > /var/www/html/index.html
```

gw の設定をします。lvs の eth2(192.168.11.117) を指定しますが、node1, node2 側から外部（インタネットー側）にアクセスすることができなくなるので注意が必要です。

```
$ sudo route add default gw 192.168.11.117
$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
192.168.11.0    0.0.0.0         255.255.255.0   U         0 0          0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth1
0.0.0.0         192.168.11.117  0.0.0.0         UG        0 0          0 eth1
```

# アクセスしてみる
```curl 192.168.0.201``` にアクセスしてみてみるべし。うまく node1, node2 にアクセスされると思います（Apacheのログを見るのもいいね）。

# 動作が不安定
なにが原因か不明なのですが、10分経過すると突然アクセスできなくなる現象があります。
まあこれきっと、vagrant のなにか？が悪さしてるんだろうな？って感じでみてますが、nat は調子悪いので DSR を使っています。





