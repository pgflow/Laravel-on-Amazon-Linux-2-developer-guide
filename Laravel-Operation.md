# <a name="pageTop"></a>Laravel Operation

作成日：2022/06/24<br>

このドキュメントは、Amazon Linux 2 に Laravel 実行環境を構築する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 目次
***
+ [Laravel update](#laravel_update)

***


## <a name="laravel_update"></a>Laravel update

ホームディレクトリへ移動し、`composer global update` を実行する。
```
cd ~
composer global update
```

Laravel プロジェクトディレクトリへ移動し、現在インストールされている Laravel のバージョンを調べます。
```
php artisan -V
```

応答
```
Laravel Framework 9.9.0
```

composer を使用して、アップデート対象を確認する。
```
composer outdated -D
```

応答
```
Info from https://repo.packagist.org: #StandWithUkraine
Color legend:
- patch or minor release available - update recommended
- major release available - update possible
guzzlehttp/guzzle 7.4.2  7.4.5   Guzzle is a PHP HTTP client library
laravel/framework v9.9.0 v4.2.22 The Laravel Framework.
laravel/jetstream v2.7.4 v2.8.5  Tailwind scaffolding for the Laravel framework.
```

対象をひとつずつ `update` する場合
```
composer update guzzlehttp/guzzle
```
```
composer update laravel/framework
```
```
composer update laravel/jetstream
```

まとめて、`update` する場合
```
composer update
```

Laravel のバージョンを調べます。
```
php artisan -V
```
応答
```
Laravel Framework 9.18.0
```