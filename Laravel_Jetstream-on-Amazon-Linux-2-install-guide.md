# <a name="pageTop"></a>Laravel Jetstream on Amazon Linux 2 install guide

作成日：2022/04/05<br>

このドキュメントは、Amazon Linux 2 に Laravel Jetstream 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ 必ず、新規の Laravel プロジェクトで Laravel Jetstream を設定してください。
+ Laravel が実行可能な事。設定方法、エラーの対応方法を確認してください。<br>-> [インストールガイド Laravel on Amazon Linux 2 install guide](Laravel-on-Amazon-Linux-2-install-guide.md) 


## 目次
***
+ [Laravel Jetstream で MariaDB を使用する準備](#setup_MariaDB_Laravel_Jetstream_db)
   + [Laravel Jetstream で使用するデータベースの作成](#setup_MariaDB_Laravel_Jetstream_db)
   + [Laravel Jetstream で使用するユーザーの作成](#setup_MariaDB_Laravel_Jetstream_user)
   + [Laravel Jetstream で使用するユーザーへ権限を付与](#setup_MariaDB_Laravel_Jetstream_user_setting)
    + [作成したユーザーで権限設定したデータベースを確認](#setup_MariaDB_Laravel_Jetstream_user_db)
***
+ [Laravel Jetstream プロジェクトの準備](#setup_laravel_jetstream_new_project)
  + [ディレクトリを作成する](#mkdir_www)
  + [Laravel Jetstream プロジェクトを作成する](#laravel_jetstream_new_project)
  + [.env ファイルへ DB_ と MAIL_ の設定](#edit_env_db_mail)
  + [nginx.conf を編集](#edit_nginxconf)
  + [Composer を使用して、Jetstream を新しい Laravel プロジェクトにインストール](#laravel_jetstream_install_project)
  + [Livewire を使用して Jetstream のインストールを完了する](#laravel_jetstream_install_livewire_project)
  + [NPM 依存関係のインストール](#npm_install)
  + [dashboard へ移動できない MbstringPHP 拡張機能のインストール](#dashboard_error_mbstring)
***

## <a name="setup_MariaDB_Laravel_Jetstream_db"></a>Laravel Jetstream で使用するデータベースの作成

MariaDB へ root でログインします。
```
mariadb -u root -p
```
パスワードが聞かれますので、設定した MariaDB の root パスワードを入力します。
```
Enter password:
```
応答
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.5.10-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```
create database "作成するデータベース名" を実行します。
```
create database laravel_jetstream_db;
```
応答
```
Query OK, 1 row affected (0.000 sec)
```

作成したデータベースを確認します。
```
show databases;
```

応答、" laravel_jetstream_db "が確認できます。
```
MariaDB [(none)]> show databases;
+----------------------+
| Database             |
+----------------------+
| information_schema   |
| laravel_db           |
| laravel_jetstream_db |
| mysql                |
| performance_schema   |
+----------------------+
5 rows in set (0.003 sec)
```

## <a name="setup_MariaDB_Laravel_Jetstream_user"></a>Laravel Jetstream で使用するユーザーの作成

localhost のみ接続できる ユーザーの作成
```
create user 'ユーザー名'@'localhost' identified by 'パスワード';
```
例
```
create user 'laravel-jetstream'@'localhost' identified by 'secretpassword';
```
何処からでも接続できる ユーザーの作成
```
create user 'ユーザー名' identified by 'パスワード'; 
```
例
```
create user 'laravel-jetstream' identified by 'secretpassword';
```

ユーザーの情報を確認する
```
select * from mysql.user where user='laravel-jetstream'\G
```

応答、ユーザーが作成されたことが確認出来ます。
```
*************************** 1. row ***************************
                  Host: %
                  User: laravel-jetstream
```

## <a name="setup_MariaDB_Laravel_Jetstream_user_setting"></a>Laravel Jetstream で使用するユーザーへデータベース レベルの権限を設定
grant all で全ての権限が対象のデータベースに対して指定したユーザーに設定されます。
```
grant all on データベース名.* to ユーザー名@接続元;
```

例：localhost のみ接続できるユーザーの設定
```
grant all on laravel_jetstream_db.* to 'laravel-jetstream'@'localhost';
```
例：何処からでも接続できる ユーザーの設定
```
grant all on laravel_jetstream_db.* to 'laravel-jetstream';
```

## <a name="setup_MariaDB_Laravel_Jetstream_user_db"></a>作成したユーザーで権限設定したデータベースを確認

root で MariaDB に入っているので、exit で一度抜けます。
```
MariaDB [(none)]> exit
Bye
```

作成したユーザで mariadb にログインします。
```
mariadb -u laravel-jetstream -p
```
ログイン出来たら、権限を設定したデータベースが見えるか確認
```
show databases;
```

応答、"laravel_jetstream_db" が表示されたので設定が出来ています。
```
MariaDB [(none)]> show databases;
+----------------------+
| Database             |
+----------------------+
| information_schema   |
| laravel_jetstream_db |
+----------------------+
2 rows in set (0.000 sec)
```

***
## <a name="setup_laravel_jetstream_new_project"></a>Laravel Jetstream プロジェクトの準備
<a name="mkdir_www"></a>/srv/ ディレクトリ内に www ディレクトリを作成する
```
cd /srv/
sudo mkdir www
ls -la
```

応答
```
drwxr-xr-x  2 root root   6 Apr  4 06:24 www
```

www ディレクトリの ユーザーとグループを変更する
```
sudo chown -R ec2-user:ec2-user www
ls -la
```

応答
```
drwxr-xr-x  2 ec2-user ec2-user   6 Apr  4 06:24 www
```

## <a name="laravel_jetstream_new_project"></a>Laravel Jetstream プロジェクトを作成する

>参考：Jetstreamのインストール<br>
https://jetstream.laravel.com/2.x/installation.html

www ディレクトリへ移動
```
cd /srv/www
```

laravel new コマンドでプロジェクトを作成する

```
laravel new アプリケーション名
```

例
```
laravel new laravel-jetstream
```

応答、（抜粋 Application ready! Build something amazing.　と表示されたらOK）
```
 _                               _
| |                             | |
| |     __ _ _ __ __ ___   _____| |
| |    / _` | '__/ _` \ \ / / _ \ |
| |___| (_| | | | (_| |\ V /  __/ |
|______\__,_|_|  \__,_| \_/ \___|_|

Creating a "laravel/laravel" project at "./laravel-jetstream"
~~~ 

~~~
Application ready! Build something amazing.
```

## <a name="edit_env_db_mail"></a>.env ファイルへ DB_ と MAIL_ の設定
作成した laravel プロジェクトディレクトリへ移動する
```
cd laravel-jetstream/
```

.env を編集します
```
nano .env
```

**> DB_**

DB_CONNECTION 変更前
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_jetstream
DB_USERNAME=root
DB_PASSWORD=
```

DB_CONNECTION 変更後
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=作成したDB名
DB_USERNAME=作成したユーザー名
DB_PASSWORD=作成したユーザー名のパスワード
```

DB_CONNECTION 変更例
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_jetstream_db
DB_USERNAME=laravel-jetstream
DB_PASSWORD=secretpassword
```

**> MAIL_**

MAIL_MAILER 変更前
```
MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

MAIL_MAILER 変更後
```
MAIL_MAILER=smtp
MAIL_HOST=SMTPサーバーアドレス
MAIL_PORT=SMTPサーバー指定ポート番号
MAIL_USERNAME=ユーザー名
MAIL_PASSWORD=パスワード
MAIL_ENCRYPTION=指定の暗号方法
MAIL_FROM_ADDRESS="送信元になるメールアドレス"
MAIL_FROM_NAME="${APP_NAME}"
```

MAIL_MAILER 変更例
```
MAIL_MAILER=smtp
MAIL_HOST=email-smtp.com
MAIL_PORT=25
MAIL_USERNAME=laravel-jetstream
MAIL_PASSWORD=secretpassword
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="laravel-jetstream@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

書き換えたら保存します。
```
・変更を保存
control + o

・File 名前を確認して Enter
File Name to Write: .env

・File を閉じます
control + x 
```

## <a name="edit_nginxconf"></a>nginx.conf を編集
[NGINX へ Laravel プロジェクトの設定準備](Laravel-on-Amazon-Linux-2-install-guide.md#nginxconf) を参考にして nginx.conf に追記してください。

例：nginx.conf
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    
#    server {
#        listen       80;
#        listen       [::]:80;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#        location = /404.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#        location = /50x.html {
#        }
#    }

    server {
        listen 80;
        listen [::]:80;

        ### ドメインネームが決まっていたらドメイン名に書き換える
        #server_name example.com;
        ### ドメイン名が決まっていない場合は、"_" と書く
        server_name _;

        ### インストールした laravel の public の場所を書く
        #root /srv/example.com/public;
        root /srv/www/laravel-jetstream/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-Content-Type-Options "nosniff";
    
        index index.php;
    
        charset utf-8;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }
    
        error_page 404 /index.php;
    
        location ~ \.php$ {
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    
        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```
[**★ ★ ★ nginx.conf のユーザーと権限の変更と、再読み込みを忘れずに行うこと ★ ★ ★**](Laravel-on-Amazon-Linux-2-install-guide.md#permission_nginxconf)

## <a name="laravel_jetstream_install_project"></a>Composer を使用して、Jetstream を新規 Laravel プロジェクトにインストール

jetstream をインストールするLaravelプロジェクトに移動します
```
cd /srv/www/laravel-jetstream/
```

Composer で Jetstream をインストールする
```
composer require laravel/jetstream
```

応答、（ Publishing complete. と表示されたらOK）
```
Using version ^2.7 for laravel/jetstream
./composer.json has been updated
Running composer update laravel/jetstream
Loading composer repositories with package information
Updating dependencies
Lock file operations: 9 installs, 0 updates, 0 removals
  - Locking bacon/bacon-qr-code (2.0.7)
  - Locking dasprid/enum (1.0.3)
  - Locking jaybizzle/crawler-detect (v1.2.111)
  - Locking jenssegers/agent (v2.6.4)
  - Locking laravel/fortify (v1.12.0)
  - Locking laravel/jetstream (v2.7.2)
  - Locking mobiledetect/mobiledetectlib (2.8.39)
  - Locking paragonie/constant_time_encoding (v2.5.0)
  - Locking pragmarx/google2fa (8.0.0)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 9 installs, 0 updates, 0 removals
  - Installing dasprid/enum (1.0.3): Extracting archive
  - Installing bacon/bacon-qr-code (2.0.7): Extracting archive
  - Installing jaybizzle/crawler-detect (v1.2.111): Extracting archive
  - Installing paragonie/constant_time_encoding (v2.5.0): Extracting archive
  - Installing pragmarx/google2fa (8.0.0): Extracting archive
  - Installing laravel/fortify (v1.12.0): Extracting archive
  - Installing mobiledetect/mobiledetectlib (2.8.39): Extracting archive
  - Installing jenssegers/agent (v2.6.4): Extracting archive
  - Installing laravel/jetstream (v2.7.2): Extracting archive
1 package suggestions were added by new dependencies, use `composer suggest` to see details.
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: jenssegers/agent
Discovered Package: laravel/fortify
Discovered Package: laravel/jetstream
Discovered Package: laravel/sail
Discovered Package: laravel/sanctum
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Discovered Package: spatie/laravel-ignition
Package manifest generated successfully.
79 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
```

## <a name="laravel_jetstream_install_livewire_project"></a>Livewire を使用して Jetstream のインストールを完了する<br>
>※ Livewire か Vue.js を選択することも出来ます。<br>
>※ --teams オプションの詳細は公式サイトで確認<br>
> https://jetstream.laravel.com/2.x/features/teams.html

Livewire を使用するオプションでインストール
```
php artisan jetstream:install livewire
```
>**注意**<br>
>--teams オプションをつけて install しても、<br>
>```
>php artisan jetstream:install livewire --teams
>```
>
>ユーザー登録後、dashboard ページで以下のエラーが出ます。<br>
>```
>ParseError<br>
>syntax error, unexpected token "@", expecting variable or "{" or "$"
>```
> 2022/04/07 installにて

応答（Livewire scaffolding installed successfully. と表示されたらOK）
```
Migration created successfully.
./composer.json has been updated
Running composer update livewire/livewire
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 0 updates, 0 removals
  - Locking livewire/livewire (v2.10.4)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing livewire/livewire (v2.10.4): Extracting archive
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: jenssegers/agent
Discovered Package: laravel/fortify
Discovered Package: laravel/jetstream
Discovered Package: laravel/sail
Discovered Package: laravel/sanctum
Discovered Package: laravel/tinker
Discovered Package: livewire/livewire
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Discovered Package: spatie/laravel-ignition
Package manifest generated successfully.
80 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
Copied Directory [/vendor/laravel/sanctum/database/migrations] To [/database/migrations]
Copied File [/vendor/laravel/sanctum/config/sanctum.php] To [/config/sanctum.php]
Publishing complete.

Livewire scaffolding installed successfully.
Please execute "npm install && npm run dev" to build your assets.
```

## <a name="npm_install"></a>NPM 依存関係のインストール
Jetstreamをインストールした後、NPM依存関係をインストールして構築し、データベースを移行する必要があります。

※以下の３つを実行します
```
npm install
npm run dev
php artisan migrate
```

npm をインストールします。
```
npm install
```

応答、npm コマンドが無い
```
-bash: npm: command not found
```

>NPM は、Node.js に含まれています<br>
**Node.js インストールの参考**<br>
Node Version Manager Installing and Updating<br>
https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating<br>
チュートリアル: Amazon EC2 インスタンスでの Node.js のセットアップ<br>
https://docs.aws.amazon.com/ja_jp/ja_jp/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html

curl で nvm（パッケージ管理） のインストールを進めます
```
cd ~
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

nvm.sh を実行します
```
. ~/.nvm/nvm.sh
```

nodejs をインストール
```
nvm install node
```

応答
```
Downloading and installing node v17.8.0...
Downloading https://nodejs.org/dist/v17.8.0/node-v17.8.0-linux-x64.tar.xz...
############################################################################################### 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v17.8.0 (npm v8.5.5)
Creating default alias: default -> node (-> v17.8.0)
```

nodejs のバージョンを確認
```
node -v
```

応答
```
v17.8.0
```

npm のバージョンを確認
```
npm -v
```

応答
```
8.5.5
```
npm のインストールが完了しました。

**> npm install を実行**

laravel-jetstream のディレクトリへ移動
```
cd /srv/www/laravel-jetstream/
```

npm install を実行
```
npm install
```

応答
```
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.

added 770 packages, and audited 771 packages in 28s

72 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

**> npm run dev を実行**
```
npm run dev
```

応答（ webpack compiled successfully 表示されたらOK ）
```
> dev
> npm run development

> development
> mix

● Mix █████████████████████████ emitting (95%)
 emit

● Mix █████████████████████████ done (99%) plugins
 WebpackBar:done

✔ Mix
  Compiled successfully in 2.92s

   Laravel Mix v6.0.43

✔ Compiled Successfully in 2886ms
┌─────────────────────────────────────────────────────────────────────────────────────────┬──────────┐
│                                                                                    File │ Size     │
├─────────────────────────────────────────────────────────────────────────────────────────┼──────────┤
│                                                                              /js/app.js │ 694 KiB  │
│                                                                             css/app.css │ 47.4 KiB │
└─────────────────────────────────────────────────────────────────────────────────────────┴──────────┘
webpack compiled successfully
```

**> php artisan migrate を実行**
```
php artisan migrate
```

応答
```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (13.87ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (15.26ms)
Migrating: 2014_10_12_200000_add_two_factor_columns_to_users_table
Migrated:  2014_10_12_200000_add_two_factor_columns_to_users_table (3.65ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (15.37ms)
Migrating: 2019_12_14_000001_create_personal_access_tokens_table
Migrated:  2019_12_14_000001_create_personal_access_tokens_table (23.28ms)
Migrating: 2022_04_07_054735_create_sessions_table
Migrated:  2022_04_07_054735_create_sessions_table (35.54ms)
```

**> Livewireスタックを使用している場合は、最初にLivewireスタックのBladeコンポーネントを公開する必要があります。**

```
php artisan vendor:publish --tag=jetstream-views
```

応答
```
Copied Directory [/vendor/laravel/jetstream/resources/views] To [/resources/views/vendor/jetstream]
Publishing complete.
```

**> 次に、にあるSVGをカスタマイズする必要があります**

```
resources/views/vendor/jetstream/components/application-logo.blade.php
resources/views/vendor/jetstream/components/authentication-card-logo.blade.php
resources/views/vendor/jetstream/components/application-mark.blade.php
```

**> これらのコンポーネントをカスタマイズした後、アセットを再構築する必要があります。 **
```
npm run dev
```

**> storage のユーザーとグループを、webサーバー のユーザー apache に変更する**
```
cd /srv/www/laravel-jetstream/
sudo chown -R apache:apache storage/
ls -la
```

**> インスタンスのパブリックIPアドレスへブラウザでアクセスし Laravel Jetstream が動いているのを確認する。**

右上の"Register"からユーザー登録すると、dashboard ページへ移動します。

![Laravel](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/Welcome_to_your_Jetstream_application.png)

## <a name="dashboard_error_mbstring"></a>dashboard へ移動できない MbstringPHP 拡張機能のインストール

ブラウザに表示される Error
```
Maximum execution time of 30 seconds exceeded
```

対処方法の参考

>サーバー要件<br>
>Laravelフレームワークにはいくつかのシステム要件があります。 <br>
>Webサーバーに次の最小PHPバージョンと拡張機能があることを確認する必要があります。 <br>
>https://laravel.com/docs/9.x/deployment#server-requirements
>
>```
>PHP> = 8.0
>BCMathPHP 拡張機能
>CtypePHP 拡張機能
>cURLPHP 拡張機能
>DOMPHP 拡張機能
>FileinfoPHP 拡張機能
>JSONPHP 拡張機能
>MbstringPHP 拡張機能
>OpenSSLPHP 拡張機能
>PCREPHP 拡張
>PDOPHP 拡張機能
>TokenizerPHP 拡張機能
>XMLPHP 拡張機能
>```
>以上のPHP拡張機能がインストールされている必要があります。
>

インストールされている php 機能拡張を確認します。
```
php -m
```


応答
```
bz2
calendar
Core
ctype
curl
date
dom
exif
fileinfo
filter
ftp
gettext
hash
iconv
json
libxml
mysqli
mysqlnd
openssl
pcntl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
readline
Reflection
session
SimpleXML
sockets
SPL
sqlite3
standard
tokenizer
xml
xmlreader
xmlwriter
xsl
zip
zlib
```

MbstringPHP 拡張機能 がインストールされていないので、install します。
```
sudo yum -y install php-mbstring
```

応答（ Complete! が表示されたらOK）
```
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                     | 3.7 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package php-mbstring.x86_64 0:8.0.16-1.amzn2 will be installed
--> Processing Dependency: libonig.so.2()(64bit) for package: php-mbstring-8.0.16-1.amzn2.x86_64
--> Running transaction check
---> Package oniguruma.x86_64 0:5.9.6-1.amzn2.0.4 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================
 Package               Arch            Version                       Repository                  Size
======================================================================================================
Installing:
 php-mbstring          x86_64          8.0.16-1.amzn2                amzn2extra-php8.0          473 k
Installing for dependencies:
 oniguruma             x86_64          5.9.6-1.amzn2.0.4             amzn2-core                 127 k

Transaction Summary
======================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 600 k
Installed size: 2.5 M
Downloading packages:
(1/2): oniguruma-5.9.6-1.amzn2.0.4.x86_64.rpm                                  | 127 kB  00:00:00
(2/2): php-mbstring-8.0.16-1.amzn2.x86_64.rpm                                  | 473 kB  00:00:00
------------------------------------------------------------------------------------------------------
Total                                                                 3.0 MB/s | 600 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : oniguruma-5.9.6-1.amzn2.0.4.x86_64                                                 1/2
  Installing : php-mbstring-8.0.16-1.amzn2.x86_64                                                 2/2
  Verifying  : php-mbstring-8.0.16-1.amzn2.x86_64                                                 1/2
  Verifying  : oniguruma-5.9.6-1.amzn2.0.4.x86_64                                                 2/2

Installed:
  php-mbstring.x86_64 0:8.0.16-1.amzn2

Dependency Installed:
  oniguruma.x86_64 0:5.9.6-1.amzn2.0.4

Complete!
```

php-fpm nginx service を再起動します

+ php-fpm サービス再起動コマンド
```
sudo systemctl restart php-fpm
```

+ NGINX サービス再起動コマンド
```
sudo systemctl restart nginx
```

もう一度、ブラウザで Laravel Jetstream の dashboard へアクセスして問題なければOK


***
+ [pageTop](#pageTop)
+ [README](README.md)
***
><a name="reference_website_php"></a>**セットアップの参考にした WebSite**<br>
>**Laravel**<br>
>+ Laravel ドキュメント<br>
https://laravel.com/docs/9.x
>+ Laravel インストーラー のセットアップ<br>
https://laravel.com/docs/9.x#the-laravel-installer
>+ laravel サーバー要件<br>
https://laravel.com/docs/9.x/deployment#server-requirements
>
>**Laravel Jetstream**<br>
> + Jetstream のインストール<br>
https://jetstream.laravel.com/2.x/installation.html
>