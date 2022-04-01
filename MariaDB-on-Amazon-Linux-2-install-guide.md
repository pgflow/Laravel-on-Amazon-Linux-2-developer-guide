# <a name="pageTop"></a>MariaDB on Amazon Linux 2 install guide

作成日：2022/03/28<br>

このドキュメントは、Amazon Linux 2 に MariaDB 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

+ [MariaDB インストール](#install_MariaDB)
+ [MariaDB 初期設定 mysql_secure_installation の実行](#mysql_secure_installation)
+ [MariaDB プロセスの自動起動設定](#enabled_MariaDB_service)
+ [MariaDB サービスコマンドリスト](#status_MariaDB_service)
+ [MariaDB インストールの参考にしたWebSite](#reference_website_MariaDB)

## <a name="install_nginx"></a>MariaDB インストール

インストールされている MariaDB のバージョンを確認する
```
mariadb -v

```

応答、MariaDB は存在していない
```
-bash: mariadb: command not found
```

amazon-linux-extras というパッケージ管理システムから MariaDB をインストールします。

パッケージ一覧を表示する
```
amazon-linux-extras
```

応答、表示されたパッケージリストに mariadb10.5 があることを確認する
```
54  mariadb10.5              available    [ =stable ]
```

MariaDB インストール
```
sudo amazon-linux-extras install mariadb10.5
``` 

応答、表示されたパッケージリストに mariadb10.5 が enabled になったことを確認する
```
 54  mariadb10.5=latest       enabled      [ =stable ]
```

MariaDB サーバーを起動します。
```
sudo systemctl start mariadb
```

MariaDB 状態
```
sudo systemctl status mariadb.service
```
応答、Active: active (running) になったことを確認する
```
● mariadb.service - MariaDB 10.5 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2022-04-01 05:21:31 UTC; 42min ago
     Docs: man:mariadbd(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 12417 (mariadbd)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─12417 /usr/libexec/mariadbd --basedir=/usr
```


インストールされている MariaDB のバージョンを確認する
```
sudo mariadb -v
```

応答
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.5.10-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Reading history-file /root/.mysql_history
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

MariaDB コンソールからログアウトします
```
exit
```
応答
```
MariaDB [(none)]> exit
Writing history-file /root/.mysql_history
Bye
```

## <a name="mysql_secure_installation"></a>MariaDB 初期設定 mysql_secure_installation の実行
https://mariadb.com/kb/ja/mysql_secure_installation/
```
sudo mysql_secure_installation
```
応答

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
```
MariaDB の現在のパスワードを入力しますが、インストールした直後はなにも設定されていないので、何も入力せずに Enter 。
```
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
すでにrootアカウントが保護されているので、安全に「n」と答えることができます。 
n と入力して Enter 。
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
```
root パスワードを設定するため、y と入力して Enter 。

```
New password:　新しいパスワードを入力
Re-enter new password:　上に入力したのと同じパスワードを入力
```
パスワードが設定された。
```
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
```
匿名ユーザーを削除するため、y と入力して Enter 。
```
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
```
root ユーザー のリモートログインを出来なくするため、y と入力して Enter 。
```
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
```
test データベースを削除するため、y と入力して Enter 。
```
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
```
特権テーブルの再読み、y と入力して Enter 。
```
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```

## <a name="enabled_MariaDB_service"></a>MariaDB プロセスの自動起動設定
Linux Server を再起動した際に、MariaDB も起動するように設定をする。

自動起動設定を確認します

```
systemctl is-enabled mariadb.service
```

応答

```
disabled
```

無効(disabled)になっているため、有効にします

```
sudo systemctl enable mariadb.service
```

応答

```
Created symlink from /etc/systemd/system/mysql.service to /usr/lib/systemd/system/mariadb.service.
Created symlink from /etc/systemd/system/mysqld.service to /usr/lib/systemd/system/mariadb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
```

自動起動設定を確認します
```
systemctl is-enabled mariadb.service
```

応答
```
enabled
```

有効になりました

## <a name="status_MariaDB_service"></a>MariaDB サービスコマンドリスト

+ MariaDB サービス状態コマンド
```
sudo systemctl status mariadb.service
```

+ MariaDB サービス起動コマンド

```
sudo systemctl start mariadb
```

+ MariaDB サービス停止コマンド

```
sudo systemctl stop mariadb
```

+ MariaDB サービス再起動コマンド

```
sudo systemctl restart mariadb
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
> <a name="reference_website_nginx"></a> **MariaDB インストールの参考にしたWebSite** <br>
Linux インスタンス用ユーザーガイド Extras library (Amazon Linux 2)<br>
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#extras-library<br>
mysql_secure_installation<br>
https://mariadb.com/kb/ja/mysql_secure_installation/
