# <a name="pageTop"></a>PHP8.0 on Amazon Linux 2 install guide

作成日：2022/03/31<br>

このドキュメントは、Amazon Linux 2 に PHP8.0 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ NGINX をインストール済みであること。<br>-> [インストールガイド NGINX on Amazon Linux 2 install guide](NGINX-on-Amazon-Linux-2-install-guide.md) 

## 目次
+ [PHP8.0 インストール](#install_php)
+ [ブラウザでアクセスし PHP が動作出来るか確認する](#welcome_php)
+ [php-fpm サービスコマンドリスト](#status_php-fpm_service)
+ [PHP8.0 インストールの参考にした WebSite](#reference_website_php)

## <a name="install_php"></a>PHP8.0 インストール

インストールされている PHP のバージョンを確認する
```
cd ~
php -v
```

応答、php は存在していない
```
-bash: php: command not found
```

amazon-linux-extras というパッケージ管理システムから php8.0 をインストールします。

パッケージ一覧を表示する
```
amazon-linux-extras
```

応答、表示されたパッケージリストに php8.0 があることを確認する
```
51  php8.0                   available    [ =stable ]
```

php8.0 インストール
```
sudo amazon-linux-extras install php8.0
``` 

応答、php-cli, php-fpm, php-mysqlnd, php-pdo, の　4 Packages が同時にインストールされます。
y と入力して Enter でインストールを進めます

```
======================================================================================================
 Package               Arch             Version                     Repository                   Size
======================================================================================================
Installing:
 php-cli               x86_64           8.0.16-1.amzn2              amzn2extra-php8.0           5.0 M
 php-fpm               x86_64           8.0.16-1.amzn2              amzn2extra-php8.0           1.7 M
 php-mysqlnd           x86_64           8.0.16-1.amzn2              amzn2extra-php8.0           189 k
 php-pdo               x86_64           8.0.16-1.amzn2              amzn2extra-php8.0           122 k
Installing for dependencies:
 libzip                x86_64           1.3.2-1.amzn2.0.1           amzn2-core                   62 k
 php-common            x86_64           8.0.16-1.amzn2              amzn2extra-php8.0           1.2 M

Transaction Summary
======================================================================================================
Install  4 Packages (+2 Dependent packages)

Total download size: 8.3 M
Installed size: 41 M
Is this ok [y/d/N]: y
```

応答、表示されたパッケージリストに php8.0 が enabled になったことを確認する
```
51  php8.0=latest            enabled      [ =stable ]
```

インストールされているPHPのバージョンを確認する
```
php -v
```

応答、PHP 8.0.16 がインストールされた
```
PHP 8.0.16 (cli) (built: Mar  1 2022 00:31:45) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.16, Copyright (c) Zend Technologies
```

php-fpm を起動します。
```
sudo systemctl start php-fpm
```

NGINX を再起動してインストールした php-fpm を適用します。
```
# sudo systemctl restart nginx
```

php-fpm 状態

```
sudo systemctl status php-fpm.service
```

応答、Active: active (running) になったことを確認する
```
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2022-03-31 04:29:20 UTC; 4min 22s ago
```

##  <a name="welcome_php"></a>ブラウザでアクセスし PHP が動作出来るか確認する

phpversion() 関数でPHPバージョンを確認するphp実行ファイルを作成し、
NGINX HTTP Server のデフォルトディレクトリへ移動します。

```
echo "<?php echo 'phpversion:'.phpversion(); ?>" > testphp.php
sudo mv testphp.php /usr/share/nginx/html/testphp.php
```

インスタンスのパブリックIPアドレスに testphp.php を付けてブラウザでアクセスし PHPのバージョンを確認する

ブラウザに以下のページが表示されたらOK
![phpversion:8.0.16](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/phptest.png)

作成した、testphp.php を削除します。
```
sudo rm /usr/share/nginx/html/testphp.php
```

## <a name="status_php-fpm_service"></a>php-fpm サービスコマンドリスト

+ php-fpm サービス状態コマンド
```
sudo systemctl status php-fpm.service
```

+ php-fpm サービス起動コマンド
```
sudo systemctl start php-fpm
```

+ php-fpm サービス停止コマンド
```
sudo systemctl stop php-fpm
```

+ php-fpm サービス再起動コマンド
```
sudo systemctl restart php-fpm
```

+ php-fpm コンフィグファイルの編集

```
sudo nano /etc/php-fpm.d/www.conf
```

+ php-fpm コンフィグファイルの再読み込み
```
sudo systemctl reload php-fpm
```

+ NGINX サービス再起動コマンド
```
sudo systemctl restart nginx
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
> <a name="reference_website_php"></a> **PHP8.0 インストールの参考にした WebSite** <br>
チュートリアル: Amazon Linux 2 に LAMP ウェブサーバーをインストールする<br>
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html<br>
Linux インスタンス用ユーザーガイド Extras library (Amazon Linux 2)<br>
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#extras-library<br>
PHP マニュアル<br>
https://www.php.net/manual/ja/
