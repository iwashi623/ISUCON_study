# ISUCON_study
ISUCONの参考記事まとめ。
コマンドはUbuntu環境。

## GitHubに公開鍵登録
```
 $ git config --global user.name "iwashi623"
 $ git config --global user.email GitHubメールアドレス
```
 - https://qiita.com/shizuma/items/2b2f873a0034839e47ce

## サーバーの状態把握
 - CPU
 ```
  $ lscpu
  または、
  $ cat /proc/cpuinfo
 ```
 
 - メモリ
 メガバイト表示、ギガバイト表示は`-g`
 ```
  $ free -m
 ```
 
 - サービス
 ```
  $ systemctl list-units --type=service
 ```

## シンボリックリンク
```
貼る　　　　　　　参照先ディレクトリ                        置く場所　　　
ln -s /home/isucon/private_isu/webapp/etc /etc/mysql/isu-mysql

剥がす.  置き場所
unlink /etc/mysql/isu-mysql
```

## 初手
 - htopインストール
 `shift + m`でメモリ使用順、`shift + P`でCPU使用順
 ```
  $ sudo apt install htop
 ```
 - file名で検索
 ```
 例）sudo find /  -name "default.conf"
 ```
 
 - systemctlでstatus確認
 ```
 sudo systemctl status nginx.service
 ```

### nginx
 - [alpのインストールとnginxのログ設定](https://nishinatoshiharu.com/install-alp-to-nginx/)
 - [色々まとまっている](https://kazegahukeba.hatenablog.com/entry/2019/09/13/015113)
 ```
 $ wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
 $ unzip alp_linux_amd64.zip

 # パスの通っているディレクトリにalpをインストール
 $ sudo install ./alp /usr/local/bin
 
 ログローテート
 # echo -n "" > /var/log/nginx/access.log && sudo chmod 777 /var/log/nginx/access.log
 過去ログ削除と再起動
 $ sudo rm /var/log/nginx/access.log && sudo systemctl reload nginx
 
 $ alp -f /var/log/nginx/access.log
 パターンで絞り込み(ここでは/posts/{post_id}と/@{hogehoge}を絞り込み)
 $ alp -f /var/log/nginx/access.log --aggregates='posts/[0-9]+,/@\w+'' --sum -r
 ```
 
 - https://github.com/Nagarei/isucon11-qualify-test/issues/1#issuecomment-912392530

### MySQL
 - MySQLのスローログ設定
 [参考](https://nishinatoshiharu.com/mysql-slow-query-log/)
 ```
 sudo vi /etc/mysql/my.cnf
 ~~~
 [mysqld]
  slow_query_log = 1
  slow_query_log_file = /var/log/mysql/slow.log
  long_query_time = 0
 ~~~
 
 sudo service mysql restart
 sudo mysql
 mysql> show variables like 'slow%';
+---------------------+-------------------------+
| Variable_name       | Value                   |
+---------------------+-------------------------+
| slow_launch_time    | 2                       |
| slow_query_log      | ON                      |
| slow_query_log_file | /var/log/mysql/slow.log |
+---------------------+-------------------------+
3 rows in set (0.01 sec)

 ```
 
  - ログローテート
  ```
  $ now=`date +%Y%m%d-%H%M%S` && sudo mv /var/log/mysql/slow.log /var/log/mysql/slow.log.$now && sudo mysqladmin flush-logs
  ```
 　
  - [pt-query-digest](https://nishinatoshiharu.com/percona-slowquerylog/)
  ```
  $ wget https://www.percona.com/downloads/percona-toolkit/3.0.10/binary/debian/xenial/x86_64/percona-toolkit_3.0.10-1.xenial_amd64.deb
  $ sudo apt-get install libdbd-mysql-perl libdbi-perl libio-socket-ssl-perl libnet-ssleay-perl libterm-readkey-perl
  $ sudo dpkg -i percona-toolkit_3.0.10-1.xenial_amd64.deb
  OR

  $ cd /usr/local/bin
  $ sudo curl -LO percona.com/get/pt-query-digest
  $ sudo chmod +x pt-query-digest
  
  $ sudo pt-query-digest /var/log/mysql/slow.log
  ```
  
  ## INDEXを貼る
  ```
  $ mysql
  ## commentsテーブルのINDEX確認
  show index from comments;
  
  ## INDEX追加
  ALTER TABLE テーブル名 ADD INDEX インデックス名(カラム名);
  
  ## 複合INDEX追加
  ALTER TABLE posts ADD INDEX index_posts_on_created_at_updated_at(created_at, updated_at);
  
  ## INDEX削除
  ALTER TABLE テーブル名 DROP INDEX インデックス名;
  ```
  
  ## log
  ```
  # systemd 
  $ sudo tail -n 1000 /var/log/syslog | grep hogehoge
  ```
