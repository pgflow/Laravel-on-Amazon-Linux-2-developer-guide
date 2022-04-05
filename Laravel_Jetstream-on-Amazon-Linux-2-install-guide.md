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

## <a name="install_Laravel"></a>Laravel インストーラー のセットアップ

または、Laravel インストーラーをグローバル Composer 依存関係としてインストールします。
```
cd ~
composer global require laravel/installer
```

応答
```
Changed current directory to /home/ec2-user/.config/composer
Info from https://repo.packagist.org: #StandWithUkraine
Using version ^4.2 for laravel/installer
./composer.json has been created
Running composer update laravel/installer
Loading composer repositories with package information
Updating dependencies
Lock file operations: 10 installs, 0 updates, 0 removals
  - Locking ~~~
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 10 installs, 0 updates, 0 removals
  - Downloading ~~~
  - Installing ~~~
6 package suggestions were added by new dependencies, use `composer suggest` to see details.
Generating autoload files
8 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

laravel コマンドをどこからでも呼び出せるように $PATH を .bash_profile へ追記します。
```
cd ~
nano .bash_profile
```

編集前の .bash_profile は以下の通り
```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
```

PATH= へ以下のパスを追記します。<br>
```
$HOME/.config/composer/vendor/bin
```

必ず " : "を先頭に付けて追記 <br>
```
PATH=$PATH:$HOME/.local/bin:$HOME/bin:$HOME/.config/composer/vendor/bin
```

編集した .bash_profile は以下の通り
```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin:$HOME/.config/composer/vendor/bin

export PATH
```

laravel を試してみます
```
laravel
```

応答
```
-bash: laravel: command not found
```

.bash_profile が適用されていないので再読み込みをします
```
source .bash_profile
```

もう一度 laravel を試してみます
```
laravel
```

応答、Laravel Installer がインストールできました。コマンドの一覧も読めます。
```
Laravel Installer 4.2.10

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display help for the given command. When no command is given display help for the list command
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi|--no-ansi  Force (or disable --no-ansi) ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  completion  Dump the shell completion script
  help        Display help for a command
  list        List commands
  new         Create a new Laravel application
```

## <a name="laravel_command"></a>laravel コマンドリスト

+ laravel アプリケーションの新規作成コマンド
```
例：laravel new example-app
```

```
laravel new 作成するアプリケーション名
```

***
## <a name="setup_MariaDB_Laravel_db"></a>Laravel で使用するデータベースの作成

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
create database laravel_db;
```
応答
```
Query OK, 1 row affected (0.000 sec)
```

作成したデータベースを確認します。
```
show databases;
```

応答、"laravel_db  "が確認できます。
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| laravel_db         |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)
```

## <a name="setup_MariaDB_Laravel_user"></a>Laravel で使用するユーザーの作成

localhost のみ接続できる ユーザーの作成
```
create user 'ユーザー名'@'localhost' identified by 'パスワード';
```
例
```
create user 'Laravel'@'localhost' identified by 'secretpassword';
```
何処からでも接続できる ユーザーの作成
```
create user 'ユーザー名' identified by 'パスワード'; 
```
例
```
create user 'Laravel' identified by 'secretpassword';
```

ユーザーの情報を確認する
```
select * from mysql.user where user='laravel'\G
```

応答、ユーザーが作成されたことが確認出来ます。
```
*************************** 1. row ***************************
                  Host: %
                  User: laravel
```

## <a name="setup_MariaDB_Laravel_user_setting"></a>Laravel で使用するユーザーへデータベース レベルの権限を設定
grant all で全ての権限が対象のデータベースに対して指定したユーザーに設定されます。
```
grant all on データベース名.* to ユーザー名@接続元;
```

例：localhost のみ接続できるユーザーの設定
```
grant all on laravel_db.* to Laravel@localhost;
```
例：何処からでも接続できる ユーザーの設定
```
grant all on laravel_db.* to laravel;
```

## <a name="setup_MariaDB_Laravel_user_db"></a>作成したユーザーで権限設定したデータベースを確認
作成したユーザで mariadb にログインします。
```
mariadb -u laravel -p
```
ログイン出来たら、権限を設定したデータベースが見えるか確認
```
show databases;
```

応答、"laravel_db" が表示されたので設定が出来ています。
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| laravel_db         |
+--------------------+
2 rows in set (0.000 sec)
```

## <a name="setup_laravel_new_project"></a>Laravel プロジェクトの準備
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

## <a name="laravel_new_project"></a>Laravel プロジェクトを作成する

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
laravel new example-app
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