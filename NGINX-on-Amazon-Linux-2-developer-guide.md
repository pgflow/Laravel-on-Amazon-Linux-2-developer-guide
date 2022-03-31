# NGINX-on-Amazon-Linux-2-developer-guide<a name="NGINX-on-Amazon-Linux-2-developer-guide"></a>

作成日：2022/03/28<br>

このドキュメントは、Amazon Linux 2 に NGINX 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

+ [NGINX インストール](#NGINX インストール)
+
+

## <a name="NGINX インストール"></a>NGINX インストール

amazon-linux-extras というパッケージ管理システムから NGINX をインストールします。

パッケージ一覧を表示する
```
amazon-linux-extras
```

応答、表示されたパッケージリストに nginx1 があることを確認する
```
38  nginx1                   available    [ =stable ]
```

NGINX インストール
```
sudo amazon-linux-extras install nginx1
``` 

応答、表示されたパッケージリストに nginx1 が enabled になったことを確認する
```
38  nginx1=latest            enabled      [ =stable ]
```

NGINX 起動
```
sudo systemctl start nginx.service
```

NGINX 状態

```
sudo systemctl status nginx.service
```

応答、Active: active (running) になったことを確認する
```
   Active: active (running) since Mon 2022-03-28 07:30:55 UTC; 4s ago
```


##  NGINX HTTP へブラウザでアクセス出来るか確認する

インスタンスのパブリックIPアドレスへブラウザでアクセスし NGINX HTTP サーバーが動いているのを確認する。

ブラウザに以下のページが表示されたらOK

![Welcome to nginx on Amazon Linux!](https://pgflow.s3.us-west-2.amazonaws.com/github/Laravel-on-Amazon-Linux-2-developer-guide/Welcome-to-nginx-on-Amazon-Linux.png)


## NGINX プロセスの自動起動設定

自動起動設定を確認します

```
systemctl is-enabled nginx.service
```

応答

```
disabled
```

無効(disabled)になっているため、有効にします

```
sudo systemctl enable nginx.service
```

応答

```
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service. 
```

自動起動設定を確認します
```
systemctl is-enabled nginx.service
```

応答
```
enabled
```

有効になりました

## NGINX サービスコマンドリスト

+ NGINX サービス状態コマンド
```
sudo systemctl status nginx.service
```

+ NGINX サービス起動コマンド

```
sudo systemctl start nginx
```

+ NGINX サービス停止コマンド

```
sudo systemctl stop nginx
```

+ NGINX サービス再起動コマンド

```
sudo systemctl restart nginx
```

+ NGINX コンフィグファイルの編集

```
sudo nano /etc/nginx/nginx.conf
```

+ NGINX コンフィグファイルの再読み込み
```
sudo systemctl reload nginx
```

***
+ [TOP - README](README.md)

***
> **NGINX インストールの参考にしたWebSite** <br>
Amazon Linux 2にExtrasレポジトリからNginxをインストールする<br>
https://dev.classmethod.jp/articles/install-nginx-on-amazon-linux2-from-extras-repository/<br>
Linux インスタンス用ユーザーガイド Extras library (Amazon Linux 2)<br>
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#extras-library

