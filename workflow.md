## ワークフローについて考察

### 作業の流れ
前提条件 AWS code commit へ repository 作成済みであること。

1. サンドボックス
    1. 作業の準備
    1. 動作するまでリモートに Git push しない
    1. 完了した 作業branch を main へマージする
    1. 告知
<br>
<br>
1. プレビュー
    1. Git から pull
    1. 動作に必要な作業
    1. 告知
<br>
<br>
1. 一般公開
    1. Git から pull
    1. 動作に必要な作業（DBに関連する物は注意）
    1. 告知
<br>
<br>

# *実際にやった作業を記録*
## サンドボックス
前提条件 Amazon Linux 2 に Laravel Jetstream が動作するまで準備できていること。

www ディレクトリへ移動
```
cd /srv/www/
```
AWS CodeCommit から git clone
```
git clone https://git-codecommit.xxx.amazonaws.com/v1/repos/xxx
```

AWS CodeCommit の HTTPS Git 認証情報を答えます
```
Username for xxx
Password for xxx
```
clone したフォルダへ移動
```
cd sandbox
```

.env ファイルを作成する
```
cp .env.example .env
```

composer の設定 サンドボックスでは `--no-dev` オプションは使用しない
```
composer install --optimize-autoloader
```

ディレクトリ、 storage/ と bootstrap/cache/ のユーザーを、webサーバー の apache に変更する
```
sudo chown -R apache:ec2-user storage/ bootstrap/cache/
```

Application キー の作成
```
php artisan key:generate
```
.env `DB_` 部分を適切な値に書き換える
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sandbox
DB_USERNAME=root
DB_PASSWORD=
```

データベースにテーブルを作成
```
php artisan migrate
```

**HeidiSQL 等を利用してデータベースに必要なデータを流し込む**


以上


.htpasswd を作成
```
printf "user:$(openssl passwd -crypt password)\n" >> .htpasswd
```

nginx.conf に追記する。
```
auth_basic "Authentication required";
auth_basic_user_file /srv/www/プロジェクトディレクトリ/.htpasswd;
```

以下のエラーへの対処方法
```
file_put_contents(/srv/www/xxx/storage/framework/views/xxx.php): Failed to open stream: Permission denied
```
`php artisan view:cache` コマンドを実行する
```
php artisan view:cache
```



### 作業の準備
1. repository から code を clone する
1. git local を設定（詳細別）

### 動作するまでリモートに Git push しない
1. git 作業ブランチ作成
1. code 編集→保存
1. git add + commit


### 完了したbranchをmainへマージする
1. git 作業ブランチ作成
1. code 編集→保存
1. git add + commit




以下のcommandスニペットをまとめる。
- Git
- MariaDB
- php artisan
- ssh
- sftp
- s3
