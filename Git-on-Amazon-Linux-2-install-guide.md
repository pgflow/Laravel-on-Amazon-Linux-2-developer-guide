# Git on Amazon Linux 2 install guide<a name="Git-on-Amazon-Linux-2-install-guide"></a>

作成日：2022/03/29<br>

このドキュメントは、Amazon Linux 2 に Git 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 目次
+ [Git インストール](#install_Git)

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

設定
```
git config --global init.defaultBranch main
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***