# <a name="pageTop"></a>Laravel on Amazon Linux 2 install guide

作成日：2022/04/01<br>

このドキュメントは、Amazon Linux 2 に Laravel 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ NGINX をインストール済みであること。<br>-> [インストールガイド NGINX on Amazon Linux 2 install guide](NGINX-on-Amazon-Linux-2-install-guide.md) 
+ PHP をインストール済みであること。<br>-> [インストールガイド PHP on Amazon Linux 2 install guide](PHP-on-Amazon-Linux-2-install-guide.md) 
+ Composer をインストール済みであること。<br>-> [インストールガイド Composer on Amazon Linux 2 install guide](Composer-on-Amazon-Linux-2-install-guide.md) 
+ MariaDB をインストール済みであること。<br>-> [インストールガイド MariaDB on Amazon Linux 2 install guide](MariaDB-on-Amazon-Linux-2-install-guide.md) 

## 目次
***
+ [Laravel インストーラー](#install_Laravel)
   + [Laravel インストーラー のセットアップ](#install_Laravel)
   + [laravel コマンドリスト](#laravel_command)
***
+ [Laravel で MariaDB を使用する準備](#setup_MariaDB_Laravel_db)
   + [Laravel で使用するデータベースの作成](#setup_MariaDB_Laravel_db)
   + [Laravel で使用するユーザーの作成](#setup_MariaDB_Laravel_user)
   + [Laravel で使用するユーザーへ権限を付与](#setup_MariaDB_Laravel_user_setting)
    + [作成したユーザーで権限設定したデータベースを確認](#setup_MariaDB_Laravel_user_db)
***
+ [Laravel プロジェクトの準備](#setup_laravel_new_project)
   + [ディレクトリを作成する](#mkdir_www)
   + [Laravel プロジェクトを作成する](#laravel_new_project)
   + [インストールのエラー解決 ext-dom extension のインストール](#laravel_new_error)
   + [Laravel プロジェクトを再度作成する](#laravel_new_project_re)
***
+ NGINX セットアップ
   + nginx.conf に Laravel server の設定を書き足す。
***

## <a name="install_Laravel"></a>Laravel インストーラー のセットアップ
https://laravel.com/docs/9.x#the-laravel-installer

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
<a name="mkdir_www"></a>user ディレクトリ内に www ディレクトリを作成する
```
cd ~
mkdir www
ls
```

応答
```
www
```

## <a name="laravel_new_project"></a>Laravel プロジェクトを作成する

www ディレクトリへ移動
```
cd www
```

laravel new コマンドでプロジェクトを作成する

```
laravel new アプリケーション名
```

例
```
laravel new example-app
```
実行すると次のインストールのエラーが表示されました、次の手順でエラーを解決します。

ext-dom エクステンションを必要としている
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
cd ~/www/
rm -rf 作成したlaravelアプリケーション名
```

例
```
cd ~/www/
rm -rf example-app
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





***
+ [pageTop](#pageTop)
+ [README](README.md)
***
> <a name="reference_website_php"></a> **セットアップの参考にした WebSite** <br>
**-Laravel-**<br>
Laravel インストーラー<br>
https://laravel.com/docs/9.x#the-laravel-installer<br>
Laravel ドキュメント<br>
https://laravel.com/docs/9.x<br>
**-MariaDB-**<br>
MariaDB入門<br>
https://www.dbonline.jp/mariadb/<br>
