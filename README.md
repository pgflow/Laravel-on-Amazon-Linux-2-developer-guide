# <a name="Laravel-on-Amazon-Linux-2-developer-guide"></a>Laravel on Amazon Linux 2 developer guide

作成日：2022/03/31<br>

このドキュメントは、Amazon Linux 2 に Laravel 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提

+ [Amazon Lightsail](https://lightsail.aws.amazon.com/)
で Amazon Linux 2 インスタンスを新規作成済みであること。<br>

+ Windows Terminal などで、SSH 接続出来る準備が出来ていること。


>Amazon Linux 2 が 5 年間の長期サポート (LTS) を付随して、正式版を公開しました。2 つの LTS candidate ビルド版を 2017 年 12 月 13 日と 2018 年 4 月 9 日にリリース<br>
>https://aws.amazon.com/jp/about-aws/whats-new/2018/06/announcing-amazon-linux-2-with-long-term-support/

>Amazon Linux 2 の長期サポートは 2023 年 6 月 30 日まで、すべての主要なパッケージのセキュリティ更新とバグ修正を提供します。<br>
>https://aws.amazon.com/jp/amazon-linux-2/faqs/

## 目次
+ [NGINX](NGINX-on-Amazon-Linux-2-install-guide.md)
+ [PHP](PHP-on-Amazon-Linux-2-install-guide.md)
+ [Composer](Composer-on-Amazon-Linux-2-install-guide.md)
+ [MariaDB](MariaDB-on-Amazon-Linux-2-install-guide.md)
+ [Git](Git-on-Amazon-Linux-2-install-guide.md)
+ [Certbot Let'sEncrypt](Certbot-LetsEncrypt-on-Amazon-Linux-2-install-guide.md)
***
+ [Laravel](Laravel-on-Amazon-Linux-2-install-guide.md)
+ [Laravel Debugbar](Laravel-Debugbar-on-Amazon-Linux-2-install-guide.md)
+ [Laravel Jetstream](Laravel_Jetstream-on-Amazon-Linux-2-install-guide.md)
+ [Laravel Jetstream Tailwind CSS guide](Laravel-Jetstream-Tailwind-CSS-guide.md)
+ [Laravel 初期設定 日本時間と日本語環境へ変更](Laravel-initial-setting.md)
+ [補足資料：Laravel 9 Import Export Excel and CSV File Tutorial](Laravel-9-Import-Export-Excel-and-CSV-File-Tutorial.md)
***
+ [Visual Studio Code](Visual_Studio_Code-on-Amazon-Linux-2-install-guide.md)
+ [Command Line Reference](Command-Line-Reference.md)

***
## 🔖Bookmarks

+ ☁️**AWS**
    + [AWS](https://aws.amazon.com/jp/)
    + [Amazon Lightsail（仮想マシン、コンテナ、データベース、CDN、ロードバランサー、DNS 管理）](https://lightsail.aws.amazon.com) 
    + [Amazon Simple Email Service（電子メールサービス）](https://aws.amazon.com/jp/ses/)
    + [Amazon Route 53（ドメインネームシステム (DNS) サービス）](https://aws.amazon.com/jp/route53/)
    + [AWS CodeCommit（プライベート Git リポジトリでのコードの保存）](https://aws.amazon.com/jp/codecommit/)

    + **Tips**
        + [スワップファイルを使用して、Amazon EC2 インスタンスのスワップ領域として機能するようにメモリを割り当てるにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-memory-swap-file/)
        + [15.2.3. スワップファイルの削除](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/storage_administration_guide/swap-removing-file)

+ 📕**Laravel**

    + [Laravel](https://laravel.com/)
    + [Laravel Jetstream](https://jetstream.laravel.com)
    + [Laravel Livewire](https://laravel-livewire.com/)
    + **Tips**
        + [laravel-news/Laravel Livewire：14のヒントとコツ](https://laravel-news.com/laravel-livewire-tips-and-tricks)

+ 🌬️**Tailwind CSS**
    + [Tailwind CSS](https://tailwindcss.com/)
    + [Tailwind Elements](https://tailwind-elements.com/)
    
+ 📬**Email**
    + [Safe Email Testing With Mailtrap’s Fake SMTP Server（偽のSMTPサーバーを使用した安全な電子メールテスト）](https://mailtrap.io/fake-smtp-server/)
    + [ProtonMail（End-to-End Encryption 及び、独自ドメインメールの受信）](https://protonmail.com/jp/)
    + [iCloud メールでカスタムメールドメインを使う](https://support.apple.com/ja-jp/HT212514)

+ 🖥️**Git**
    + **Git**
        + [Git](https://git-scm.com/)
        + [Pro Git book](https://git-scm.com/book/)
    + **GitHub**
        + [GitHub](https://github.com/)
        + [Git Guide (GitHub)](https://github.com/git-guides)
        + [GitHub Desktop](https://desktop.github.com/)
    + **Bitbucket**
        + [Bitbucket](https://bitbucket.org/)
        + [Sourcetree](https://www.sourcetreeapp.com/)
    + **AWS CodeCommit**
        + [AWS CodeCommit（プライベート Git リポジトリでのコードの保存）](https://aws.amazon.com/jp/codecommit/)
        + [AWS CodeCommit のドキュメント](https://docs.aws.amazon.com/ja_jp/codecommit/index.html)
+ 🗃️**Database**
    + [Data Types in MariaDB](https://mariadb.com/kb/en/data-types/)
    + [HeidiSQL OpenSource Database client](https://www.heidisql.com/)
