# Git on Amazon Linux 2 install guide<a name="Git-on-Amazon-Linux-2-install-guide"></a>

作成日：2022/03/29<br>

このドキュメントは、Amazon Linux 2 に Git 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 目次
+ [Git インストール](#install_Git)
+ [最初のGitの構成](#gitconfig)
+ [プロジェクト毎のGitの構成](#project_gitconfig)

## <a name="install_Git"></a>Git インストール
Git がインストール済みか確認する
```
git version
```

応答、Git コマンドが存在しない
```
-bash: git: command not found
```

Git インストール amazon-linux-extras に Git が無いので、yum を利用します
```
sudo yum install git
``` 

応答、最終行に Complete! が表示されたらインストール完了
```
Complete!
```

Git がインストール済みか再度確認する
```
git version
```

応答、バージョンが表示されたらインストール出来ています。
```
git version 2.32.0
```

## <a name="gitconfig"></a>Git 最初のGitの構成
ホームディレクトリへ移動しておきます。
```
cd
```
応答
```
[ec2-user@ip-xxx-xx-x-xxx ~]$
```
書式設定と空白文字への対応を設定します。<br>
（Linux や Mac の場合）※lightsail に Amazon Linux 2 instance を作成して作業する場合です。
```
git config --global core.autocrlf input
```

デフォルトブランチ名を ```main``` に設定
```
git config --global init.defaultBranch main
```

個人の識別情報を設定
```
git config --global user.name "your name"
```
```
git config --global user.email your@mailaddress
```

設定の確認
```
git config --list
```

応答
```
core.autocrlf=input
init.defaultbranch=main
user.name=your name
user.email=your@mailaddress
```

## <a name="project_gitconfig"></a>プロジェクト毎のGitの構成

このステップには、AWS CodeCommit に作成した空のリポジトリURLとIAM ユーザー用の Git 認証情報が必要です。

プロジェクトディレクトリへ移動します。
```
cd /srv/www/example-app
```

既存のディレクトリでのリポジトリの初期化
```
git init
```

応答
```
Initialized empty Git repository in /srv/www/example-app/.git/
```

 .git という名前の新しいサブディレクトリが作られます。

 ```
 ls -la
total 0
drwxrwxr-x  3 ec2-user ec2-user  18 Jun  3 06:33 .
drwxr-xr-x 11 ec2-user ec2-user 162 Jun  3 06:32 ..
drwxrwxr-x  7 ec2-user ec2-user 119 Jun  3 06:33 .git
 ```


デフォルトブランチ名を ```main``` に設定
```
git config init.defaultBranch main
```

個人の識別情報を設定
```
git config user.name "company your name"
```
```
git config user.email company.your@mailaddress
```

リモートリポジトリの追加
※AWS CodeCommit に作成した空のリポジトリURLが必要です。
```
git remote add origin https://git-codecommit.ap-northeast-x.amazonaws.com/v1/repos/example-app
```

最初のファイルを作成して、```fast``` と書いて保存します。※ファイル名や内容は何でもよい。
```
nano fast.txt
```
```control + O``` で保存し、 ```control + X``` で閉じます。

バージョン管理を始めます。
```
git add fast.txt
```
コメントを付けて、コミットします。
```
git commit -m 'initial project version'
```
応答
```
[main 62f90dc] initial project version
 1 file changed, 1 insertion(+), 1 deletion(-)
```

変更を ```push``` します。
```
git push --set-upstream origin main
```

Username を入力（IAM ユーザー用の Git 認証情報）
```
Username for 'https://git-codecommit.ap-northeast-1.amazonaws.com':
```

Password を入力（IAM ユーザー用の Git 認証情報）
```
Password for 'https://pymana-at-055966712913@git-codecommit.ap-northeast-1.amazonaws.com':
```

応答
```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 260 bytes | 260.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/example-app
   ba7a2ed..62f90dc  main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

設定を確認します。
```
git config --list
```

応答
```
core.autocrlf=input
init.defaultbranch=main
user.name=
user.email=
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
init.defaultbranch=main
user.name=company your name
user.email=company.your@mailaddress
remote.origin.url=https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/example-app
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
```

AWS CodeCommit にログインし、リポジトリの変更が反映されているか確認してください。

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
>Linux や Mac などの行末に LF を使うシステムで作業をしている場合は、Git にチェックアウト時の自動変換をされてしまうと困ります。しかし、行末が CRLF なファイルが紛れ込んでしまった場合には Git に自動修正してもらいたいものです。 コミット時の CRLF から LF への変換はさせたいけれどもそれ以外の自動変換が不要な場合は、core.autocrlf を input に設定します。<br>
>改行コードの問題<br>
https://www.git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%81%AE%E8%A8%AD%E5%AE%9A#_core_autocrlf

>git initを実行した後、マスターブランチの名前を変更するのを忘れないでください。 このオプションはあなたを助けます！ <br>
https://www.seancdavis.com/posts/git-set-default-branch/