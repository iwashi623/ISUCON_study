# ISUCON_study
ISUCONの参考記事まとめ。
コマンドはUbuntu環境。

## GitHubに公開鍵登録
```
 $ git config --global user.name "SardineTa23"
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
  $ systemctl list units --type=service
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
 
 過去ログ削除と再起動
 $ sudo rm /var/log/nginx/access.log && sudo systemctl reload nginx
 
 $ alp -f /var/log/nginx/access.log
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
  
  $ sudo pt-query-digest /var/log/mysql/slow.log
  ```
