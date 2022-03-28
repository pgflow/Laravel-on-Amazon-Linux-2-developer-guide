.md


# NGINX-on-Amazon-Linux-2-developer-guide<a name="NGINX-on-Amazon-Linux-2-developer-guide"></a>

作成日：2022/03/28<br>

このドキュメントは、Amazon Linux 2 に NGINX 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

* amazon-linux-extras というパッケージ管理システムから NGINX をインストールします。



```
    function test() {
        let a = 0;
        }

    test
        test
```


>nginx インストールの参考にしたWebSite<br>
https://dev.classmethod.jp/articles/install-nginx-on-amazon-linux2-from-extras-repository/


パッケージ一覧

``` $ amazon-linux-extras ```

nginx インストール

``` $ sudo amazon-linux-extras install nginx1 ``` 


nginx 起動

``` $ sudo systemctl start nginx.service ```


nginx 状態

``` $ sudo systemctl status nginx.service ```


インスタンスのIPアドレスへブラウザでアクセスし nginx HTTP サーバーが動いているのを確認する。

接続先ドメインの例

camellia.pgflow.click

Nginxプロセスの自動起動設定

自動起動設定を確認します

``` $ systemctl is-enabled nginx.service ```

応答

``` disabled ```

無効(disabled)になっているため、有効にします

```
sudo systemctl enable nginx.service
```

応答

```
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service. 
```

自動起動設定を確認します

$ systemctl is-enabled nginx.service

応答

enabled

有効になりました


nginx サービスコマンド

nginx サービス状態コマンド

$ sudo systemctl status nginx.service

nginx サービス起動コマンド

$ sudo systemctl start nginx

nginx サービス停止コマンド

$ sudo systemctl stop nginx

nginx サービス再起動コマンド

$ sudo systemctl restart nginx