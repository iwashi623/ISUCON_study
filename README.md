# ISUCON_study
ISUCONの参考記事まとめ

## 初手
 - file名で検索
 ```
 例）sudo find /  -name "default.conf"
 ```
 - [alpのインストールとnginxのログ設定](https://nishinatoshiharu.com/install-alp-to-nginx/)
 ```
 過去ログ削除と再起動
 sudo rm /var/log/nginx/access.log && sudo systemctl reload nginx
 ```
 
 - https://github.com/Nagarei/isucon11-qualify-test/issues/1#issuecomment-912392530

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
 　
  - [pt-query-digest](https://nishinatoshiharu.com/percona-slowquerylog/)_
