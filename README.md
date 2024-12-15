# isucon-my-sheet

ISUCONの練習で使ったコマンドや参考になった記事のまとめ

## 環境構築

### ssh しようとしたときに使った秘密鍵で、BadPermisshion のエラーが出た時

秘密鍵のパーミッションを 400 にする
```bash
$ chmod 400 ~/.ssh/秘密鍵名.pem
```

### isucon ユーザーでローカルから SSH する

isucon ユーザーの~/.ssh/authorized_keys に Github に登録している公開鍵を追加
https://github.com/iwashi623.keys

```bash
$ mkdir .ssh && touch ~/.ssh/authorized_keys
```

### ssh-agent とローカルの秘密鍵を利用して、isucon サーバーから GitHub の認証を通す

```bash
以下、ローカル
--------------------------------------
**秘密鍵をssh-agentに登録**
$ ssh-add -k ~/.ssh/使いたい秘密鍵
**秘密鍵が登録されたかの確認**
$ ssh-add -l
**ssh-agentを使ってインスタンスへssh**
ssh -A  -i ~/.ssh/使いたい秘密鍵 isucon@<インスタンスIP>

以下、インスタンス内
--------------------------------------
成功すると以下のように表示される
$ ssh git@github.com
PTY allocation request failed on channel 0
Hi iwashi623! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

### GitHub に公開鍵登録(新しい秘密を作りたい時以外は不要)

- https://qiita.com/shizuma/items/2b2f873a0034839e47ce

```bash
 $ git config --global user.name "iwashi623"
 $ git config --global user.email GitHubメールアドレス
```

### Git コマンドのエイリアスを作成

```bash
$ vi ~/.gitconfig

---------以下を貼り付け---------
[user]
    name = iwashi623
    email = <githubメールアドレス>

[alias]
    s = status
    ci = commit
    l = log --oneline
    co = checkout
    br = branch
```

## サーバーの状態把握
- CPU

```bash
 $ lscpu
 または、
 $ cat /proc/cpuinfo
```

- メモリ
  メガバイト表示、ギガバイト表示は`-g`

```bash
 $ free -m
```

- ストレージ
```bash
 $ df -hT
```

- サービス
```bash
 $ systemctl list-units --type=service
```

### シンボリックリンク
```bash
貼る　　　　　　　参照先ディレクトリ                        置く場所　　　
ln -s /home/isucon/private_isu/webapp/etc /etc/mysql/isu-mysql

剥がす.  置き場所
unlink /etc/mysql/isu-mysql
```

### scp
```bash
ローカルのファイルを転送
$ scp webapp/mysql/db/2_DummyChairData.sql isucon@<IPアドレス>:/home/isucon/isuumo/webapp/mysql/db/

リモートのディレクトリを転送
$ scp -r isucon@<IPアドレス>:/home/isucon/isuumo/webapp/mysql/db webapp/mysql/ 
```

### 環境把握
- htop インストール
```bash
$ sudo apt install htop
```
`shift + m`でメモリ使用順、`shift + P`で CPU 使用順でソートできる

- 開いているPortの確認
```bash
$ sudo netstat -anp
```

- file 名で検索
```bash
例）sudo find /  -name "default.conf"
```

- systemd の設定ファイルを探す
```bash
$ sudo find / -type f -name "*isucari*"

直接コマンドで設定ファイルも見れる
$ sudo systemctl cat isucari.go.service
```

- systemctl で status 確認
```bash
sudo systemctl status nginx.service
```

サービスを停止(再起動しても立ち上がらない)
```bash
sudo systemctl disable isuride-matcher
```

### Systemd に環境変数を渡す

```bash
$ vi /etc/systemd/system/isucari.golang.service
[Unit]
Description=My Go Application

[Service]
ExecStart=/path/to/your/go-binary
Environment="VAR1=value1" "VAR2=value2" <- ここを編集する
Restart=always
User=username
Group=groupname

[Install]
WantedBy=multi-user.target



$ sudo systemctl daemon-reload && sudo systemctl start isucari.golang && sudo systemctl enable isucari.golang
```

### systemdの設定ファイルを書き換えたら
```bash
設定ファイルの読み込み
$ systemctl daemon-reload
```

### アプリを restart したい時

```bash
sudo systemctl disable --now isucholar.go.service
sudo systemctl enable --now isucholar.go.service
sudo systemctl status isucholar.go.service
```
### ローカルホストから､指定したホスト上でコマンドを実行して､結果をクリップボードへ
```bash
ssh -A i1 "cd webapp && make nalp" | pbcopy
```

## nginx
### 設定確認
```bash
# エラーログなどを確認できる
sudo nginx -t
```

### alp

- https://github.com/tkuchiki/alp
- [alp のインストールと nginx のログ設定](https://nishinatoshiharu.com/install-alp-to-nginx/)
- [色々まとまっている](https://kazegahukeba.hatenablog.com/entry/2019/09/13/015113)

```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

worker_cpu_affinity auto;

# nginx worker の設定
worker_rlimit_nofile  32768;
events {
  worker_connections  8096;  # 128より大きくするなら、 max connection 数を増やす必要あり。さらに大きくするなら worker_rlimit_nofile も大きくする（file descriptor数の制限を緩める)
  multi_accept on;         # 複数acceptを有効化する
  use epoll; # 待受の利用メソッドを指定（基本は自動指定されてるはず）
}

http {
	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	  log_format ltsv "time:$time_local"
	    "\thost:$remote_addr"
	    "\tforwardedfor:$http_x_forwarded_for"
	    "\treq:$request"
	    "\tmethod:$request_method"
	    "\turi:$request_uri"
	    "\tstatus:$status"
	    "\tsize:$body_bytes_sent"
	    "\treferer:$http_referer"
	    "\tua:$http_user_agent"
	    "\treqtime:$request_time"
	    "\truntime:$upstream_http_x_runtime"
	    "\tapptime:$upstream_response_time"
	    "\tcache:$upstream_http_x_cache"
	    "\tvhost:$host";

        
        error_log /var/log/nginx/error.log;
        access_log  /var/log/nginx/access.log ltsv;

  	gzip on;

	  # gzip_vary on;
	  # gzip_proxied any;
	  # gzip_comp_level 6;
	  # gzip_buffers 16 8k;
	  # gzip_http_version 1.1;
	  # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # 基本設定
    sendfile    on;
    tcp_nopush  on;
    tcp_nodelay on;
    types_hash_max_size 2048;
    server_tokens    off;
    open_file_cache max=100 inactive=20s;

    proxy_buffers 100 32k;
    proxy_buffer_size 8k;

    # mime.type の設定
    include       /etc/nginx/mime.types;
	   default_type application/octet-stream;

    # Keepalive 設定
    # ベンチマークとの相性次第ではkeepalive off;にしたほうがいい
    # keepalive off;

    keepalive_requests 1000000;
    keepalive_timeout 600s;

    http2_max_requests 1000000;
    http2_recv_timeout 600s;

    # オリジンから来るCache-Controlを無視する必要があるなら。。。
    #proxy_ignore_headers Cache-Control;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

```bash
$ wget https://github.com/tkuchiki/alp/releases/download/v1.0.21/alp_linux_amd64.zip
$ unzip alp_linux_amd64.zip
$ sudo install ./alp /usr/local/bin
```

ログのローテーション

```bash
$ echo -n "" > /var/log/nginx/access.log && sudo chmod 777 /var/log/nginx/access.log
過去ログ削除と再起動
$ sudo rm /var/log/nginx/access.log && sudo systemctl reload nginx
```

実行

```bash
$ alp -f /var/log/nginx/access.log
パターンで絞り込み(ここでは/posts/{post_id}と/@{hogehoge}を絞り込み)
$ alp -f /var/log/nginx/access.log --aggregates='posts/[0-9]+,/@\w+'' --sum -r
$ alp --sum -r -f /var/log/nginx/access.log --aggregates='/api/estate/[0-9]+,/api/chair/[0-9]+,/api/recommended_estate/[0-9]+' > hoge.txt
```

参考

- https://github.com/Nagarei/isucon11-qualify-test/issues/1#issuecomment-912392530

### レスポンスをキャッシュする
[参考](https://github.com/oystersjp/yubikega-isucon10-qualify/commit/2868646c550bd3e8680e38aa5aa6579986dccf03)
```bash
proxy_cache_path /var/cache/nginx keys_zone=zone1:1m max_size=100m inactive=5m;

server {
    root /home/isucon/isucon10-qualify/webapp/public;
    listen 80 default_server;
    listen [::]:80 default_server;

    location ~ ^/api/estate/[0-9]+$ {
            proxy_cache       zone1;
            proxy_cache_key   $scheme$proxy_host$uri$is_args$args;
            proxy_cache_valid 200 30s;
            proxy_pass http://localhost:1323;
    }

    location /api/estate/search {
            proxy_cache       zone1;
            proxy_cache_key   $scheme$proxy_host$uri$is_args$args;
            proxy_cache_valid 200 30s;
            proxy_pass http://localhost:1323;
    }

    location /api {
            proxy_pass http://localhost:1323;
    }

    location /initialize {
            proxy_pass http://localhost:1323;
    }

    location / {
            root /www/data;
    }
}
```

### 問答無用で500エラーを返す
```bash
if ($http_user_agent ~* ^isubot ) {
  return 503;
}
```

### 特定のパスだけ､別のホストに流す
```bash
map $request_method$request_uri $backend {
    default          http://127.0.0.1:8000;
    ~^POST/login$    http://172.31.26.130:8000;
}

server {
    listen 443 ssl;
    server_name isucari.*;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass $backend;
    }
}
```

## MySQL

### スキーマの確認
```bash
$ sudo mysqldump --compact --no-data isuconp | grep -v "^SET" | grep -v "^/\*\!" | perl -ple 's@CREATE TABLE @\nCREATE TABLE @g'
```

### pt-query-digest

- MySQL のスローログ設定
  [参考](https://nishinatoshiharu.com/mysql-slow-query-log/)

```bash
sudo vi /etc/mysql/my.cnf
~~~
[mysqld]
 max_connections=10000
 slow_query_log = 1
 slow_query_log_file = /var/log/mysql/slow.log
 long_query_time = 0
 disable-log-bin
 innodb_buffer_pool_size = 2GB # ディスクイメージをメモリ上にバッファさせる値をきめる設定値
 innodb_flush_log_at_trx_commit = 2 # 1に設定するとトランザクション単位でログを出力するが 2 を指定すると1秒間に1回ログファイルに出力するようになる
 innodb_flush_method = O_DIRECT # データファイル、ログファイルの読み書き方式を指定する(実験する価値はある)
 innodb_doublewrite = 0
 innodb_log_writer_threads = off
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

```bash
$ now=`date +%Y%m%d-%H%M%S` && sudo mv /var/log/mysql/slow.log /var/log/mysql/slow.log.$now && sudo mysqladmin flush-logs
```



- [pt-query-digest](https://nishinatoshiharu.com/percona-slowquerylog/)

```bash
$ cd /usr/local/bin
$ sudo curl -LO percona.com/get/pt-query-digest
$ sudo chmod +x pt-query-digest

$ sudo pt-query-digest /var/log/mysql/slow.log
```

### INDEX を貼る

```sql
$ mysql
## tableの詳細確認
show create table table_name;

## commentsテーブルのINDEX確認
show index from comments;

## INDEX追加
ALTER TABLE テーブル名 ADD INDEX インデックス名(カラム名);

## 複合INDEX追加
ALTER TABLE posts ADD INDEX index_posts_on_created_at_updated_at(created_at, updated_at);

## INDEX削除
ALTER TABLE テーブル名 DROP INDEX インデックス名;
```

### Trigger
```sql
CREATE TRIGGER update_chair_total_distances AFTER INSERT ON chair_locations FOR EACH ROW
BEGIN
	DECLARE distance INTEGER;
	-- 最新の累計距離を計算
	SET distance = IFNULL(
		(SELECT ABS(NEW.latitude - latitude) + ABS(NEW.longitude - longitude)
		FROM chair_locations
		WHERE chair_id = NEW.chair_id
		ORDER BY created_at DESC
		LIMIT 1, 1),
		0
	);
	-- 累計距離テーブルを更新または挿入
	INSERT INTO chair_total_distances (chair_id, total_distance, updated_at)
	VALUES (NEW.chair_id, distance, NEW.created_at)
	ON DUPLICATE KEY UPDATE
		total_distance = total_distance + VALUES(total_distance),
		updated_at = VALUES(updated_at);
END;
```

### DB分割
#### 移行先DBホスト
1. ユーザーと権限の追加
```sql
-- ユーザーの作成
CREATE USER 'isucon'@'%' IDENTIFIED BY 'isucon';
-- 権限の追加
GRANT ALL PRIVILEGES ON `database_name`.* TO 'isucon'@'%';

-- DBのユーザー一覧取得
SELECT user, host FROM mysql.user;
-- DBのユーザー権限確認('isucon'@'172.31.%'はユーザー名)
SHOW GRANTS FOR 'isucon'@'%';
```
2. bind-addressを0.0.0.0にする
```
$ sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0 <- こうする
```

#### アプリケーションホスト
```bash
// 環境変数設定を確認
$ sudo systemctl cat isuumo.go
```
DBのHostの設定をDBのHostにしたいサーバーのPrivateIPアドレスに切り替える｡アプリで使っている箇所があればそちらも切り替える

## pprof
### 導入
1. [こんな感じにアプリに入れる](https://github.com/hoge-times/isucon9-qualify-20240217/pull/5)
2. ベンチ実行後､Initializeが終わったタイミングで以下のコマンド事項
   ```bash
    $ go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
   ```
3. ローカルでファイルを展開する
   ```bash
    $ go tool pprof -http 127.0.0.1:9090 ./pprof.isucari.samples.cpu.002.pb.gz
   ```

## log

```bash
$ journalctl -xe | grep  isucari
$ sudo journalctl -u isuports.service -b
$ sudo journalctl -u isuports.service -b -r #逆順

# systemd
# 行数を固定して表示
$ sudo tail -n 1000 /var/log/syslog | grep hogehoge
# 流れてくるログをみる
$ sudo tail -f /var/log/syslog
```

## (練習用)Goのバージョンを変える
```bash
goのパスを確認しておく
$ which go
$ printenv | grep GOROOT

goのパッケージを取ってくる
$ wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz

既存のGo削除
$ rm -rf /home/isucon/local/go

取ってきたGoのパッケージを解凍してGoコマンドが読みに行くPathに展開する
$ tar -C /home/isucon/local/ -xzf go1.21.5.linux-amd64.tar.gz
```
