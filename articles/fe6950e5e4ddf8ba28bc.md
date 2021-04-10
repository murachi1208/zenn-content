---
title: "Munin で複数サーバのグラフを１つにまとめて表示す"
emoji: "📝"
type: "tech"
topics: Munin Vagrant CentOS6.5
published: true
---

# 複数サーバのグラフを１つにまとめたい
仮想環境は簡単に作成できて使い勝手もいいのですが沢山作りすぎてしまい仮想とはいえ個別に見るのは辛い。
で複数サーバのグラフを１つにまとめて見ることができれば楽なのでその設定方法。そしていつものごとく、少しはまる…というおち（笑）。
↓こんな感じで、2サーバ混ぜてみたｗ
![munin_05.png](https://qiita-image-store.s3.amazonaws.com/0/44540/40860f0e-99c6-19ea-b6ad-f920839f28ea.png)

# graph_order でグループ単ににまとめる(だけ)
設定してもグラフが真っ白だったので、調べたら単にプラグインとプラグインの値の指定を間違えていただけというオチでした。

```
hoge.graph_order \
  redmine_gr=redmine:load.load \
  gitlab_gr=gitlab:load.load
```

たとえば上記の「redmine_gr=:load.load」場合
　redmine_gr : ラベル
　redmine    : muninで定義しているホスト名（実ホスト名ではない） 
　load.load  : load プラグインの load プロパティ

> [munin日本語情報ノート サンプル/グラフ引数のサンプル](http://munin.jp/wiki/%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB/%E3%82%B0%E3%83%A9%E3%83%95%E5%BC%95%E6%95%B0%E3%81%AE%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB)

## プラグインのプロパティを調べる方法は、２通り。
### munin-run コマンドを使って調べる方法
plugins ディレクトリにインストールされているプラグイン名を、munin-run コマンドの引数に指定するだけ。下記の場合、load.value の値を使用したい場合は、load プロパティが使用可能だとわかる（って説明がクドイ）。

```
# munin-run load
load.value 0.22
```

### telnet コマンドを使って調べる方法
4949 ポートにアクセスしてみる。list コマンドと fetch コマンドで簡単に確認できる。

```
# telnet 192.168.11.200 4949
Trying 192.168.11.200...
Connected to 192.168.11.200.
Escape character is '^]'.
# munin node at gitlab
list
apache_accesses apache_processes apache_volume cpu df df_inode entropy forks fw_packets if_err_eth0 if_eth0 interrupts irqstats load memory netstat nfs4_client nfs_client ntp_kernel_err ntp_kernel_pll_freq ntp_kernel_pll_off ntp_offset ntp_states open_files open_inodes postfix_mailqueue postfix_mailvolume proc_pri processes swap threads uptime users vmstat
fetch load
load.value 0.00
.
```

# 設定の確認
確認作業は、Munin が node 情報を収集する間隔（５分）で行われるため多少の待ちが必要。ただ graph_order コマンドで描画されるグラフの元データは、すでに作成されたグラフ情報と重ねるだけなので指定がうまいこといけばすぐに期待とおりの結果をみることができる。

# 参考サイト

> [東京電力の電力使用状況をmuninにしてみる](http://pi.blog.jp/archives/51703574.html)
> [Loaning data from other graphs](http://munin-monitoring.org/wiki/LoaningData)
> [muninの設定(つづき)](http://d.hatena.ne.jp/over80/20080703/1215100867)
> [munin 4949](https://www.google.co.jp/webhp?sourceid=chrome-instant&ion=1&espv=2&es_th=1&ie=UTF-8#q=munin%204949)
> [Muninでアラートメールの送信 (メール送信編)](http://castor.s26.xrea.com/blog/2007/10/19)
> [Munin/設定](http://www.sssc.cc/wiki/index.php?Munin%2F%E8%A8%AD%E5%AE%9A)





