# <a name="pageTop"></a>Composer on Amazon Linux 2 install guide

作成日：2022/03/31<br>

このドキュメントは、Amazon Linux 2 に Composer 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ PHP をインストール済みであること。<br>-> [インストールガイド PHP8.0 on Amazon Linux 2 install guide](PHP-on-Amazon-Linux-2-install-guide.md) 

## 目次
+ [Composer インストール](#install_Composer)

## <a name="install_Composer"></a>Composer インストール

インストールされている Composer のバージョンを確認する
```
composer -v
```

応答、composer は存在していない
```
-bash: composer: command not found
```

Composer インストール<br>
※必ず公式サイトの最新のインストールコマンドを使用してください。<br>
[Command-line installation](https://getcomposer.org/download/)
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '906a84df04cea2aa72f40b5f787e49f22d4c2f19492ac310e8cba5b96ac8b64115ac402c8cd292b8a03482574915d1a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

応答、ダウンロードが終わり、php composer.phar が実行されます。
```
All settings correct for using Composer
Downloading...

Composer (version 2.3.2) successfully installed to: /home/ec2-user/composer.phar
Use it: php composer.phar
```

php -r "unlink('composer-setup.php');" を実行します。(Enter)
```
php -r "unlink('composer-setup.php');"
```

composer.phar を /usr/local/bin/ へ 名称を composer にして移動する
```
sudo mv composer.phar /usr/local/bin/composer
```

インストールされている Composer のバージョンを確認する
```
composer -v
```

応答、Composer 2.3.2 がインストールされた
```
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer 2.3.2 2022-03-30 20:45:25
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
> <a name="reference_website_php"></a> **Composer インストールの参考にした WebSite** <br>
Composer<br>
https://getcomposer.org/
