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
+ [Laravel Jetstream プロジェクトの準備](#setup_laravel_Jetstream_new_project)
   + [ディレクトリを作成する](#mkdir_www)
   + [Laravel Jetstream プロジェクトを作成する](#laravel_Jetstream_new_project)
   + [インストールのエラー解決 ext-dom extension のインストール](#laravel_Jetstream_new_error)
   + [Laravel Jetstream プロジェクトを再度作成する](#laravel_Jetstream_new_project_re)
***



+ [NGINX へ Laravel プロジェクトの設定準備](#nginxconf)
  + [sftp 接続して nginx.conf をダウンロード](#sftp_get_nginxconf)
  + [nginx.conf を編集 Laravel server の設定を書き足す](#edit_nginxconf)
  + [sftp 接続して nginx.conf をアップロード](#sftp_put_nginxconf)
  + [nginx.conf のユーザーと権限の変更](#permission_nginxconf)
***
+ [Laravel storage ディレクトリの権限変更](#permission_laravel_storage)
***

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

## <a name="setup_MariaDB_Laravel_Jetstream_user"></a>Laravel で使用するユーザーの作成

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

## <a name="setup_MariaDB_Laravel_user_setting"></a>Laravel で使用するユーザーへデータベース レベルの権限を設定
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

## <a name="setup_MariaDB_Laravel_user_db"></a>作成したユーザーで権限設定したデータベースを確認

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


作成した laravel プロジェクトディレクトリへ移動する
```
cd laravel-jetstream/
```

## .env ファイルの DB_CONNECTION の設定を編集する
```
nano .env
```

変更前
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_jetstream
DB_USERNAME=root
DB_PASSWORD=
```

変更後
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=作成したDB名
DB_USERNAME=作成したユーザー名
DB_PASSWORD=作成したユーザー名のパスワード
```

変更例
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_jetstream_db
DB_USERNAME=laravel-jetstream
DB_PASSWORD=secretpassword
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

## nginx.conf を編集
[NGINX へ Laravel プロジェクトの設定準備](Laravel-on-Amazon-Linux-2-install-guide.md#a-name"nginxconf"anginx-へ-laravel-プロジェクトの設定準備)を参考にして nginx.conf に 追記してください。

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

    ### server_name が未設定な場合で有効にしたい時は 他の server {} より先に書きます。
    
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

先に進み前に、ブラウザでインストールした Laravel を確認します。<br>
必ず忘れずに行う<br>
※ nginx.conf の再読み込み<br>
※ storage ディレクトリの権限変更


## Composerを使用して、Jetstreamを新しいLaravelプロジェクトにインストール

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
Info from https://repo.packagist.org: #StandWithUkraine
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

## ＃Livewire を使用してJetstreamをインストールする<br>
※ Livewire か Vue.js を選択することも出来ます。<br>
※ --teams オプションの詳細は > https://jetstream.laravel.com/2.x/features/teams.html

Livewire を使用して、teams を利用する場合のインストール
```
php artisan jetstream:install livewire --teams
```

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
  - Downloading livewire/livewire (v2.10.4)
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


## NPM依存関係のインストール

＃
インストールの完了

Jetstreamをインストールした後、NPM依存関係をインストールして構築し、データベースを移行する必要があります。
```
npm install
```

応答、npm コマンドが無い
```
-bash: npm: command not found
```


>https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating


>https://docs.aws.amazon.com/ja_jp/ja_jp/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html


```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```


```
. ~/.nvm/nvm.sh
```

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

## npm のインストールが完了したので、再度 npm install を実行
laravel-jetstream のディレクトリへ移動
```
cd /srv/www/laravel-jetstream/
```
**npm install を実行**
```
npm install
```

応答
```
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.

added 772 packages, and audited 773 packages in 48s

72 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 8.5.5 -> 8.6.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.6.0
npm notice Run npm install -g npm@8.6.0 to update!
npm notice
```

**npm run dev を実行**
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

✔ Compiled Successfully in 2893ms
┌─────────────────────────────────────────────────────────────────────────────────────────┬──────────┐
│                                                                                    File │ Size     │
├─────────────────────────────────────────────────────────────────────────────────────────┼──────────┤
│                                                                              /js/app.js │ 694 KiB  │
│                                                                             css/app.css │ 47.4 KiB │
└─────────────────────────────────────────────────────────────────────────────────────────┴──────────┘
webpack compiled successfully
```

**php artisan migrate**
```
php artisan migrate
```

応答
```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (12.98ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (11.73ms)
Migrating: 2014_10_12_200000_add_two_factor_columns_to_users_table
Migrated:  2014_10_12_200000_add_two_factor_columns_to_users_table (3.56ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (11.22ms)
Migrating: 2019_12_14_000001_create_personal_access_tokens_table
Migrated:  2019_12_14_000001_create_personal_access_tokens_table (18.02ms)
Migrating: 2020_05_21_100000_create_teams_table
Migrated:  2020_05_21_100000_create_teams_table (12.52ms)
Migrating: 2020_05_21_200000_create_team_user_table
Migrated:  2020_05_21_200000_create_team_user_table (12.41ms)
Migrating: 2020_05_21_300000_create_team_invitations_table
Migrated:  2020_05_21_300000_create_team_invitations_table (36.01ms)
Migrating: 2022_04_06_065254_create_sessions_table
Migrated:  2022_04_06_065254_create_sessions_table (28.16ms)
```

>不要
storage のpermission を確認
storage の権限をec2-userに戻す
sudo chown -R ec2-user:ec2-user storage/
ls -la
storage の権限をapacheに変更する
sudo chown -R apache:apache storage/
ls -la
>不要

storage のユーザーとグループを、webサーバー のユーザー apache に変更する
```
cd /srv/www/laravel-jetstream/
sudo chown -R apache:apache storage/
ls -la
```



データベースにユーザーを追加する
```
```
npm run dev
php artisan migrate
```



実行するとインストールのエラー ext-dom エクステンションが必要だとする内容が表示されました、次の手順でエラーを解決します。
```
 _                               _
| |                             | |
| |     __ _ _ __ __ ___   _____| |
| |    / _` | '__/ _` \ \ / / _ \ |
| |___| (_| | | | (_| |\ V /  __/ |
|______\__,_|_|  \__,_| \_/ \___|_|

Creating a "laravel/laravel" project at "./example-app"
Info from https://repo.packagist.org: #StandWithUkraine
Installing laravel/laravel (v9.1.3)
  - Downloading laravel/laravel (v9.1.3)
  - Installing laravel/laravel (v9.1.3): Extracting archive
Created project in /home/ec2-user/www/example-app
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - phpunit/phpunit[9.5.10, ..., 9.5.x-dev] require ext-dom * -> it is missing from your system. Install or enable PHP's dom extension.
    - Root composer.json requires phpunit/phpunit ^9.5.10 -> satisfiable by phpunit/phpunit[9.5.10, ..., 9.5.x-dev].

To enable extensions, verify that they are enabled in your .ini files:
    - /etc/php.ini
    - /etc/php.d/20-bz2.ini
    - /etc/php.d/20-calendar.ini
    - /etc/php.d/20-ctype.ini
    - /etc/php.d/20-curl.ini
    - /etc/php.d/20-exif.ini
    - /etc/php.d/20-fileinfo.ini
    - /etc/php.d/20-ftp.ini
    - /etc/php.d/20-gettext.ini
    - /etc/php.d/20-iconv.ini
    - /etc/php.d/20-mysqlnd.ini
    - /etc/php.d/20-pdo.ini
    - /etc/php.d/20-phar.ini
    - /etc/php.d/20-sockets.ini
    - /etc/php.d/20-sqlite3.ini
    - /etc/php.d/20-tokenizer.ini
    - /etc/php.d/20-zip.ini
    - /etc/php.d/30-mysqli.ini
    - /etc/php.d/30-pdo_mysql.ini
    - /etc/php.d/30-pdo_sqlite.ini
You can also run `php --ini` in a terminal to see which files are used by PHP in CLI mode.
Alternatively, you can run Composer with `--ignore-platform-req=ext-dom` to temporarily ignore these required extensions.
```

## <a name="laravel_new_error"></a>インストールのエラー解決 ext-dom extension のインストール

ext-dom extension は php-xml に含まれるので、php-xml をインストール
>How do I install the dom extension for PHP7?<br>
https://laracasts.com/discuss/channels/servers/how-do-i-install-the-dom-extension-for-php7
```
sudo yum install php-xml
```

応答
```
Dependencies Resolved

======================================================================================================
 Package            Arch              Version                      Repository                    Size
======================================================================================================
Installing:
 php-xml            x86_64            8.0.16-1.amzn2               amzn2extra-php8.0            173 k
Installing for dependencies:
 libxslt            x86_64            1.1.28-6.amzn2               amzn2-core                   240 k

Transaction Summary
======================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 413 k
Installed size: 1.2 M
```
y と入力してインストールを続行します。
```
Is this ok [y/d/N]: y
```

応答
```
Downloading packages:
(1/2): libxslt-1.1.28-6.amzn2.x86_64.rpm                                       | 240 kB  00:00:00
(2/2): php-xml-8.0.16-1.amzn2.x86_64.rpm                                       | 173 kB  00:00:00
------------------------------------------------------------------------------------------------------
Total                                                                 2.1 MB/s | 413 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libxslt-1.1.28-6.amzn2.x86_64                                                      1/2
  Installing : php-xml-8.0.16-1.amzn2.x86_64                                                      2/2
  Verifying  : php-xml-8.0.16-1.amzn2.x86_64                                                      1/2
  Verifying  : libxslt-1.1.28-6.amzn2.x86_64                                                      2/2

Installed:
  php-xml.x86_64 0:8.0.16-1.amzn2

Dependency Installed:
  libxslt.x86_64 0:1.1.28-6.amzn2

Complete!
```

## <a name="laravel_new_project_re"></a>Laravel プロジェクトを再度作成する
作成に失敗した laravel プロジェクトを削除します。
```
cd /srv/www/
rm -rf 作成したlaravelアプリケーション名
```

例
```
cd /srv/www/
rm -rf example-app
```

laravel を作成する
```
cd /srv/www/
laravel new example-app
```

応答
```
 _                               _
| |                             | |
| |     __ _ _ __ __ ___   _____| |
| |    / _` | '__/ _` \ \ / / _ \ |
| |___| (_| | | | (_| |\ V /  __/ |
|______\__,_|_|  \__,_| \_/ \___|_|

Creating a "laravel/laravel" project at "./example-app"
Info from https://repo.packagist.org: #StandWithUkraine
Installing laravel/laravel (v9.1.3)
  - Installing laravel/laravel (v9.1.3): Extracting archive
Created project in /home/ec2-user/www/example-app
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies
Lock file operations: 108 installs, 0 updates, 0 removals
  - Locking brick/math (0.9.3)
~~~
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 108 installs, 0 updates, 0 removals
  - Installing doctrine/inflector (2.0.4): Extracting archive
~~~
71 package suggestions were added by new dependencies, use `composer suggest` to see details.
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: laravel/sail
Discovered Package: laravel/sanctum
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Discovered Package: spatie/laravel-ignition
Package manifest generated successfully.
78 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
> @php artisan key:generate --ansi
Application key set successfully.

Application ready! Build something amazing.
```
laravel プロジェクトが正常に作成されました。

## <a neme="nginxconf"></a>NGINX へ Laravel プロジェクトの設定準備

既存の nginx.conf ファイルをコピーして Laravel に必要な設定を書き足して、書き換えた、nginx.conf に置き換えます。

元に戻せるように、nginx.conf をコピーしておきます。
```
cd /etc/nginx/
sudo cp nginx.conf nginx.conf.cp
```

nginx.conf を、user ディレクトリへコピーする
```
cp nginx.conf /home/ec2-user/nginx.conf
```

## <a neme="sftp_get_nginxconf"></a>sftp 接続して nginx.conf をダウンロード

新しい Terminal ウィンドウを開きます。<br>
※Windowsであれば、command プロンプト、powershell、Windows Terminal を開きます。<br>※macOS の場合は、新しい Terminal を開きます。

接続方法の説明
```
sftp -i キーファイル.pem ユーザー名@パブリックIPv4
```

例
```
sftp -i .\.ssh\LightsailDefaultKey-ap-northeast-1.pem ec2-user@192.168.0.100
```

応答、sftp> と表示されたのを確認。
```
sftp>
```

nginx.conf があるか確認します。
```
cd /home/ec2-user
ls -la
```

応答
```
-rw-r--r--    1 ec2-user ec2-user     3853 Apr  5 06:31 nginx.conf
```

get コマンドで、nginx.conf を指定します。
```
get nginx.conf
```

sftp 接続したウィンドウへ<br>
ダウンロード先フォルダをドロップするとファイルパスが入ります。
```
get nginx.conf C:\Users\ユーザー名\Downloads
```

例
```
get nginx.conf C:\Users\username\Downloads
```

上記コマンドを Enter すると<br>
自分のPCのダウンロードフォルダに nginx.conf が保存されました。
![nginx.conf](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/nginx_conf_download.png)


## <a neme="edit_nginxconf"></a>nginx.conf を編集 Laravel server の設定を書き足す

Visual Studio Code 等で nginx.conf を開きます。

既存の server { をコメントアウトします。
```
    # server {
    #     listen       80;
    #     listen       [::]:80;
    #     server_name  _;
    #     root         /usr/share/nginx/html;

    #     # Load configuration files for the default server block.
    #     include /etc/nginx/default.d/*.conf;

    #     error_page 404 /404.html;
    #     location = /404.html {
    #     }

    #     error_page 500 502 503 504 /50x.html;
    #     location = /50x.html {
    #     }
    # }
```

この行の後に、新しい server { 設定を追記します。
```
    include /etc/nginx/conf.d/*.conf;
```

以下の内容を書き換えて追記します。
>常に最新の内容を参考にしてください。<br>
laravel サーバー構成 Nginx<br>
https://laravel.com/docs/9.x/deployment#nginx

変更箇所の説明
```
server {
    listen 80;
    listen [::]:80;

    ### ドメインネームが決まっていたらドメイン名に書き換える
    #server_name example.com;
    ### ドメイン名が決まっていない場合は、"_" と書く
    server_name _;

    ### インストールした laravel の public の場所を書く
    #root /srv/example.com/public;
    root /srv/www/example-app/public;
    
    ### START nginx デフォルト設定を読み込む設定 >（phpに影響します）
    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;
    ### -END- nginx デフォルト設定を読み込む設定

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
        ### コメントアウト※nginx デフォルト設定を読み込むため
        #fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        #fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

書き換えた内容は以下の通り、nginx.conf を保存します。
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

    server {
        listen 80;
        listen [::]:80;

        ### ドメインネームが決まっていたらドメイン名に書き換える
        #server_name example.com;
        ### ドメイン名が決まっていない場合は、"_" と書く
        server_name _;

        ### インストールした laravel の public の場所を書く
        #root /srv/example.com/public;
        root /srv/www/example-app/public;
        
        ### START nginx デフォルト設定を読み込む設定 >（phpに影響します）
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        ### -END- nginx デフォルト設定を読み込む設定

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
            ### コメントアウト※nginx デフォルト設定を読み込むため
            #fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
            #fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }
    
        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

    # server {
    #     listen       80;
    #     listen       [::]:80;
    #     server_name  _;
    #     root         /usr/share/nginx/html;

    #     # Load configuration files for the default server block.
    #     include /etc/nginx/default.d/*.conf;

    #     error_page 404 /404.html;
    #     location = /404.html {
    #     }

    #     error_page 500 502 503 504 /50x.html;
    #     location = /50x.html {
    #     }
    # }

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

## <a neme="sftp_put_nginxconf"></a>sftp 接続して nginx.conf をアップロード
sftp 接続したウィンドウで、
ユーザーディレクトリへ移動し、put コマンドを入力
```
cd /home/ec2-user
put 
```

編集した nginx.conf をドロップすると以下のようにファイルパスが入力されるので、Enter 。
```
put C:\Users\username\Downloads\nginx.conf
```

応答、100% でアップロード完了
```
Uploading C:/Users/username/Downloads/nginx.conf to /home/ec2-user/nginx.conf
C:/Users/username/Downloads/nginx.conf                                                      100% 3955   247.4KB/s   00:00
```

## <a neme="permission_nginxconf"></a>nginx.conf のユーザーと権限の変更
**ssh 接続した Terminal window に変更**<br>
ユーザーディレクトリへアップロードした nginx.conf を移動する
```
cd ~
sudo mv nginx.conf /etc/nginx/nginx.conf
```

変更して移動した nginx.conf のユーザーと権限を確認して変更します
```
cd /etc/nginx/
ls -la
```

応答
```
-rw-rw-r--  1 ec2-user ec2-user 3853 Apr  4 06:40 nginx.conf
```

以下のコマンドで権限を変更します。
```
sudo chown -R root:root nginx.conf
sudo chmod 644 nginx.conf
```

変更を確認します
```
cd /etc/nginx/
ls -la
```

応答
```
-rw-r--r--  1 root root 3853 Apr  4 06:40 nginx.conf
```

NGINX コンフィグファイルの再読み込みをして nginx.conf の変更を適用します。
```
sudo systemctl reload nginx
```

Laravel へブラウザでアクセス出来るか確認する

インスタンスのパブリックIPアドレスへブラウザでアクセスし Laravel が動いているのを確認する。

ブラウザに以下の以下のエラーが表示されます
![laravel_storage_permission](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/laravel_storage_permission.png)

エラー抜粋<br>
Laravel storage ディレクトリの権限変更 が必要、次の手順で変更します。
```
The stream or file "/srv/www/example-app/storage/logs/laravel.log" could not be opened in append mode: Failed to open stream: Permission denied The exception occurred while attempting to log
```

## <a neme="permission_laravel_storage"></a>Laravel storage ディレクトリの権限変更
storage 以下のファイルに webサーバー apache から書き込みが出来ないためエラーが出ている、現在の権限を確認

```
cd /srv/www/example-app/
ls -la
```

応答 storage ディレクトリのユーザーとグループが webサーバー apache ではなく、ec2-user になっている。
```
drwxrwxr-x  5 ec2-user ec2-user     46 Mar 29 14:48 storage
```

storage のユーザーとグループを、webサーバー のユーザー apache に変更する
```
cd /srv/www/example-app/
sudo chown -R apache:apache storage/
ls -la
```

応答
```
drwxrwxr-x  5 apache   apache       46 Mar 29 14:48 storage
```
変更出来た。

もう一度 インスタンスのパブリックIPアドレスへブラウザでアクセスし Laravel が動いているのを確認する。
![Laravel](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/Laravel_beginning.png)

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
>+ laravel サーバー構成 Nginx<br>
https://laravel.com/docs/9.x/deployment#nginx
>
>**MariaDB**<br>
> + MariaDB入門<br>
https://www.dbonline.jp/mariadb/
>
>**SFTP**<br>
>+ sftp コマンドでファイルを転送する方法<br>
https://dezanari.com/ssh-sftp/#toc3
>+ sftp（Filezilla）を使用して Amazon Lightsail の Linux または UNIX インスタンスに接続する<br>
https://lightsail.aws.amazon.com/ls/docs/ja_jp/articles/amazon-lightsail-connecting-to-linux-unix-instance-using-sftp