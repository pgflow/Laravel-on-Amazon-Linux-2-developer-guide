# <a name="pageTop"></a>Laravel Debugbar on Amazon Linux 2 install guide

作成日：2022/04/27<br>

このドキュメントは、Amazon Linux 2 に Laravel Debugbar 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ 公開用 Laravel プロジェクトで Laravel Debugbar を設定しないでください。
+ Laravel が実行可能な事。設定方法、エラーの対応方法を確認してください。<br>-> [インストールガイド Laravel on Amazon Linux 2 install guide](Laravel-on-Amazon-Linux-2-install-guide.md) 

## 目次
***
+ [Laravel Debugbar](#laravel_debugbar)
   + [Laravel Debugbar のセットアップ](#set_up_laravel_debugbar)
   + [laravel Debugbar の非表示](#laravel_debugbar_false)
***

## <a name="laravel_debugbar"></a>Laravel Debugbar インストーラー
Debugbar をインストールする Laravel プロジェクトディレクトリへ移動します。
```
cd /srv/www/laravel
```

## <a name="set_up_laravel_debugbar"></a>Laravel Debugbar のセットアップ
```composer``` を使用してインストールします。
```
composer require barryvdh/laravel-debugbar --dev
```

config/debugbat.php を作成します。
```
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

## <a name="laravel_debugbar_false"></a>Laravel Debugbar の非表示
Debugbarは ```.env``` の ```APP_DEBUG=true``` で表示される。<br>
```APP_DEBUG=false``` で表示を止められます。
```
APP_DEBUG=false
```

また、Debugbarは ```.env``` の ```APP_DEBUG=true``` のままでも、<br>
```DEBUGBAR_ENABLED=false``` を書き足す事で表示を止められます。
```
DEBUGBAR_ENABLED=false
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
>Laravel Debugbar<br>
>https://github.com/barryvdh/laravel-debugbar<br>
>Laravelのエコシステム〜その２<br>
>https://www.public.ne.jp/2021/04/07/laravel%E3%81%AE%E3%82%A8%E3%82%B3%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%80%9C%E3%81%9D%E3%81%AE%EF%BC%92/