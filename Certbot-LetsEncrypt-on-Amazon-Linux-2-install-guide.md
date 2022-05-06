# <a name="pageTop"></a>Certbot Lets'Encrypt on Amazon Linux 2 install guide

作成日：2022/05/06<br>

このドキュメントは、Amazon Linux 2 にインストールした NGINX で ホストしている WebSite に HTTPS（暗号化通信）を利用出来るよう環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ NGINX を使用して HTTP(80ポート)で WebSiteをホストしてること。<br>-> [インストールガイド NGINX on Amazon Linux 2 install guide](NGINX-on-Amazon-Linux-2-install-guide.md) 

## 目次
+ [PHP8.0 インストール](#install_php)


+ [PHP8.0 インストール](#install_php)
+ [ブラウザでアクセスし PHP が動作出来るか確認する](#welcome_php)
+ [php-fpm サービスコマンドリスト](#status_php-fpm_service)
+ [PHP8.0 インストールの参考にした WebSite](#reference_website_php)





## nginx.conf へ独自ドメインを設定する


```
sudo nano /etc/nginx/nginx.conf
```

```server_name  _;``` と書いている所の ```_``` を、独自ドメインに書き換えます。

変更前
```
    server {
        server_name  _;
```

変更後（```example.com``` を使用するドメイン名に読み替えてください）
```
    server {
        server_name  example.com;
```

