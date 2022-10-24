# <a id="pageTop"></a> NGINX AUTH BASIC set up（ベーシック認証）


作成日：2022/08/22

このドキュメントは、NGINX にベーシック認証を設定する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提

+ NGINX をインストール済みであること。<br>-> [インストールガイド NGINX on Amazon Linux 2 install guide](NGINX-on-Amazon-Linux-2-install-guide.md)

## 目次

+ [httpd-tools をインストール](#install_httpd-tools)
+ [.htpasswd ファイルを作成](#htpasswd)
+ [nginx.conf に設定を追加](#nginxconf)

***

## <a id="install_httpd-tools"></a>httpd-tools をインストール

httpd-tools をインストールして、.htpasswd ファイルを作成する。


```bash
sudo yum install httpd-tools
```

あるいは、以下のコマンドでインストールする。

```bash
```bash
yum install httpd-tools
```

応答

```

Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                       | 3.7 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package httpd-tools.x86_64 0:2.4.54-1.amzn2 will be installed
--> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-tools-2.4.54-1.amzn2.x86_64
--> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-tools-2.4.54-1.amzn2.x86_64
--> Running transaction check
---> Package apr.x86_64 0:1.7.0-9.amzn2 will be installed
---> Package apr-util.x86_64 0:1.6.1-5.amzn2.0.2 will be installed
--> Processing Dependency: apr-util-bdb(x86-64) = 1.6.1-5.amzn2.0.2 for package: apr-util-1.6.1-5.amzn2.0.2.x86_64
--> Running transaction check
---> Package apr-util-bdb.x86_64 0:1.6.1-5.amzn2.0.2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================
 Package                      Arch                   Version                           Repository                  Size
========================================================================================================================
Installing:
 httpd-tools                  x86_64                 2.4.54-1.amzn2                    amzn2-core                  88 k
Installing for dependencies:
 apr                          x86_64                 1.7.0-9.amzn2                     amzn2-core                 122 k
 apr-util                     x86_64                 1.6.1-5.amzn2.0.2                 amzn2-core                  99 k
 apr-util-bdb                 x86_64                 1.6.1-5.amzn2.0.2                 amzn2-core                  19 k

Transaction Summary
========================================================================================================================
Install  1 Package (+3 Dependent packages)

Total download size: 328 k
Installed size: 641 k
Is this ok [y/d/N]:

```

y を入力してインストールを実行します。


応答

```

Downloading packages:
(1/4): apr-1.7.0-9.amzn2.x86_64.rpm                                                              | 122 kB  00:00:00
(2/4): apr-util-1.6.1-5.amzn2.0.2.x86_64.rpm                                                     |  99 kB  00:00:00
(3/4): apr-util-bdb-1.6.1-5.amzn2.0.2.x86_64.rpm                                                 |  19 kB  00:00:00
(4/4): httpd-tools-2.4.54-1.amzn2.x86_64.rpm                                                     |  88 kB  00:00:00
------------------------------------------------------------------------------------------------------------------------
Total                                                                                   1.7 MB/s | 328 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : apr-1.7.0-9.amzn2.x86_64                                                                             1/4
  Installing : apr-util-bdb-1.6.1-5.amzn2.0.2.x86_64                                                                2/4
  Installing : apr-util-1.6.1-5.amzn2.0.2.x86_64                                                                    3/4
  Installing : httpd-tools-2.4.54-1.amzn2.x86_64                                                                    4/4
  Verifying  : apr-util-1.6.1-5.amzn2.0.2.x86_64                                                                    1/4
  Verifying  : apr-util-bdb-1.6.1-5.amzn2.0.2.x86_64                                                                2/4
  Verifying  : apr-1.7.0-9.amzn2.x86_64                                                                             3/4
  Verifying  : httpd-tools-2.4.54-1.amzn2.x86_64                                                                    4/4

Installed:
  httpd-tools.x86_64 0:2.4.54-1.amzn2

Dependency Installed:
  apr.x86_64 0:1.7.0-9.amzn2      apr-util.x86_64 0:1.6.1-5.amzn2.0.2      apr-util-bdb.x86_64 0:1.6.1-5.amzn2.0.2

Complete!

```

## <a id="htpasswd"></a>.htpasswd ファイルを作成
Git で .htpasswd ファイルを管理されないようにする為、プロジェクトディレクトリへ移動し、.gitignore に .htpasswd を追記します。

```

cd /srv/www/your_project

```

nano で .gitignore を開き、.htpasswd を追加します。

```
nano .gitignore

```

応答

```
/node_modules
/public/hot
/public/storage
/storage/*.key
/vendor
.env
.env.backup
.phpunit.result.cache
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
/.idea
/.vscode

```

.htpasswd を追記します。

```

.htpasswd

```

`control` + `x` で保存して終了します。

.htpasswd ファイルを作成します。
`-c` オプションは最初に作成する際にパスワードを入力するためのオプションです。
ユーザ名とパスワードを追加する場合は、`-c` オプションを指定しないでください。

新規の場合

```bash
htpasswd -c .htpasswd your_user_name
```

ユーザーを追加する場合

```bash
htpasswd .htpasswd your_user_name
```

応答

```bash
New password:＜パスワードを入力＞
Re-type new password:＜パスワードを確認入力＞
Adding password for user your_user_name
```

cat コマンドで .htpasswd ファイルの内容を確認します。

```bash
cat .htpasswd
```

応答

```bash
your_user_name:$apr1$J8CsFlwf$2ZXOHw9l3k5V/DS.EyxcO1
```

## <a id="nginxconf"></a>nginx.conf に設定を追加 nginx.conf

nginx.conf に auth_basic と auth_basic_user_file を追加します。

```bash
sudo nano /etc/nginx/nginx.conf
```

server { } の中に以下の設定を追加します。

```bash
auth_basic "Authentication required";
auth_basic_user_file /srv/www/your_project/.htpasswd;
```

```bash
server {
        listen 80;
        listen [::]:80;
        server_name your_domain_name;
        root /srv/www/your_project/public;

        # basic認証時のメッセージダイアログ
        auth_basic "Authentication required";
        # .htpasswd ファイルのパスを指定
        auth_basic_user_file /srv/www/your_project/.htpasswd;
```

`control` + `x` で保存して終了します。

nginx を再起動します。

```bash
sudo systemctl restart nginx
```

ブラウザで your_domain_name にアクセスし、basic 認証が有効になっていることを確認します。

***

+ [pageTop](#pageTop)
+ [README](README.md)

***

> nginxでベーシック（Basic）認証を設定する方法  
<https://www.server-memo.net/server-setting/nginx/basic_auth.html>
