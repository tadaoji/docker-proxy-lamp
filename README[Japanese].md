# README.md: docker-proxy-lamp

# ToC
# 概要
nginxのproxy（steveltn/https-portal）を使って1ホストに対し複数のwebサーバを立ち上げる。ここではLet's encryptを用いSSL化されたLAMP構成を立ち上げる。デフォルトではphp-apache、mysql、wordpress、phpmyadminを自動化し構築する。

# 必要な環境
Linux（CentOS7で検証している）
docker-compose
docker

# 設定方法
docker-compose.ymlを編集する
一まとまりのセクションを##############（comment）で区切ってある

## https-portal
[SteveLTN/https-portal : https://github.com/SteveLTN/https-portal](https://github.com/SteveLTN/https-portal)
を利用している

### restart: always
動作をテストする時にはコメントアウトすることを推奨する。
alwaysの場合、なんらかの原因で再起動がかかり続けた場合毎回Let's encryptでSSL発行を試みるため、Let's encryptのSSL発行リクエスト制限に引っかかる場合がある。

### environment
#### DOMAINS
proxyの設定を行う。
基本的な書式は以下の通りである。
この例では2つのwebサーバ（php-apache_1とwordpress-app_1）に接続する。
```
'test.example.fqdn -> http://php-apache_1, wp-test.example.fqdn -> http://wordpress-app_1'
```
webサーバ名の指定はservice名に一致する。
同じnetworkに指定されているため、DNSの解決はservice名で行われるためである。

#### STAGE
productionの場合、Let's encryptにより使用可能なSSLが発行される。
localの場合、自己証明書になる。
テストの際にはLet's encryptの回数上限に達するのを回避するため、localを推奨する。
詳しくは、https://github.com/SteveLTN/https-portal を参照のこと。

### depends_on
Let's encryptによるSSL発行が行われるので、webサーバが立ち上がった状態でproxyを立ち上げる。もしあなたがサービスを追加して立ち上げるのであればここに追記することを推奨する。

## LAMP(php-apache, mysql-db)
このセクションを任意に増やすことで任意の数のlampサーバを作ることが出来る。
増やす場合、コピーした上で以下の部分を重複しないよう書き換える
php-apache
* service名
* volumes（ホスト側）
* depends_on

mysql-db
* service名
* volumes（ホスト側）
* volumesセクションにmysql-dbで使用するvolumeを追加

### php-apache
phpのオフィシャルdocker imageを利用している
[php : https://hub.docker.com/_/php](https://hub.docker.com/_/php)

#### build
##### args
DOCKER_LOCATIONを指定することでコンテナ内で動作するPHPのtimezoneを設定することができる。これは作成されるコンテナ内の/usr/share/zoneinfo/以下のディレクトリ構造の一致する。
記述しない場合はUTCになる。

#### volumes
公開ディレクトリを./lamp_1/public_htmlに設定する。
ログディレクトリを./lamp_1/apache_logsに設定する。
これらは必要に応じて書き換えることができる。

#### networks
これはすべてのコンテナで共通の設定にする必要がある。

### mysql-db
mysqlにはversion 5:Latestが使われる。

#### build
##### args
DOCKER_LOCATIONを指定することでコンテナ内で動作するPHPのtimezoneを設定することができる。これは作成されるコンテナ内の/usr/share/zoneinfo/以下のディレクトリ構造の一致する。
記述しない場合はUTCになる。

MYSQL_ROOT_PASSWORDを設定するとコンテナ内に/root/.my.cnfが作られる。これによってmysqlコマンドをコンテナ内で使用する際パスワードが不要になる。

#### environment
任意の設定が可能である。


## phpmyadmin
### https-portal-phpmyadmin
#### ports
phpmyadminの性質上、一般に公開する必要がないため、portはランダムなものを推奨する。
任意のportのみで運用するためhttps-portalもlamp用とは別にコンテナを作成する。

#### environment
##### STAGE
80,443共に使用していないので、Let's encryptの認証が通らない。
そのため自己証明書を用いるものとしてlocalを使用している。

### phpmyadmin
[https://hub.docker.com/r/phpmyadmin/phpmyadmin/](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
上記イメージを使用する。

#### environment
phpmyadminの接続するDBを固定するか決めることができる
PMA_ARBITARYを1に設定することで、ログイン時に任意のサーバを選ぶことができるようになる
PMA_HOSTを任意のDBにすることで固定状態になる。
詳しくは[https://hub.docker.com/r/phpmyadmin/phpmyadmin/](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)

#### ログイン方法
ログイン画面でDB接続先を指定する場合はDBのservice名を入力する。
service名はDocker networkの機能によりDNS解決が行われる。

# Dockerfile
## ファイル構成
```
├── docker-lamp
│   ├── mysql
│   │   └── Dockerfile
│   └── php-apache
│       └── Dockerfile
```

## mysql
### general_log
mysqlのgeneral_logをコンテナ内からファイルに出力させる場合、以下のコメントアウトを外す。
```
# general_log=1\n\  # logファイルが大きくなるので注意
# general_log_file=/var/log/mysql/query.log\n\  # logファイルが大きくなるので注意
```
この設定はコンテナ内の/etc/mysql/my.cnfに書き込まれる。
ただし、ファイルが大きくなる点に注意が必要である。

dockerのvolumeオプションにてlogファイルをホスト側から扱えるようにする際はディレクトリのpermissionに77xが必要になる場合がある。この変更を行わない場合、mysqlはlogファイルの書き込みエラーが発生しコンテナがダウンする。
# Credits
[https-portal by SteveLTN](https://github.com/SteveLTN)
[phpmyadmin by Docker Official Images](https://hub.docker.com/_/phpmyadmin)
[php by Docker Official Images](https://hub.docker.com/_/php)
[mysql by Docker Official Images](https://hub.docker.com/_/mysql)