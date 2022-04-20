# <a name="pageTop"></a>**Laravel Jetstream Tailwind CSS guide**

作成日：2022/04/20<br>

このドキュメントは、Laravel Jetstream に インストール されている TailwindCSS 実行環境を構築して利用する事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提
+ Laravel をインストール済みであること。<br>-> [インストールガイド Laravel on Amazon Linux 2 install guide](Laravel-on-Amazon-Linux-2-install-guide.md) 
+ Laravel に Jetstream をインストール済みであること。<br>-> [インストールガイド Laravel Jetstream on Amazon Linux 2 install guide](Laravel_Jetstream-on-Amazon-Linux-2-install-guide.md) 

## 目次
+ [Tailwind CSS 概要](#tailwindcss)
    + [Tailwind CSS コンパイル 概要](#tailwindcss_compile)
    + [Tailwind CSS コンパイル コマンド](#tailwindcss_compile_command)
    + [ブラウザのキャッシュを削除](#tailwindcss_browser_clear_cache)
    + [開発中の WebSite からローカルにキャッシュを保存させない](#tailwindcss_cache_control_no_cache)
    + [welcome.blade.php で Tailwind CSS を使用する](#tailwindcss_welcome_blade_php)
    + [max_user_watches 変更対象ファイルの監視上限数を変更する必要がある場合](#max_user_watches)
***

## <a name="tailwindcss"></a>**Tailwind CSS 概要**

※ **Laravel Jetstream は、Tailwind CSS がインストールされています。</br>
以下の手順ですぐに実行できます。**

Tailwind CSSは、すべてのHTMLファイル、JavaScriptコンポーネント、およびその他のテンプレートでクラス名をスキャンし、対応するスタイルを生成して、静的CSSファイルに書き込むことで機能します。 <br>
>Get started with Tailwind CSS<br>https://tailwindcss.com/docs/installation

Tailwind CSS は、HTMLファイルの html tag に書いた、class="xxx" CSS スタイルの変更を見つけ、CSSとJS を独自にコンパイルして css と js ファイルを作成します。
```
例：<h1 class="xxx">TailwindCSS</h1>
例：テキストサイズを大きくしている：<h1 class="text-9xl">TailwindCSS</h1>
```
コンパイルされ作成されたCSS と JS は以下の public フォルダに保存されます。
```
/public/css/app.css
/public/js/app.js
```
※Tailwind CSS で編集した場合必ず再コンパイルが必要です。

※ コンパイルし作成された app.css および app.js はブラウザにキャッシュされるため、Tailwind CSS を利用して編集変更しても HTMLファイルが反映されないように見える場合があります。


## <a name="tailwindcss_compile"></a>**Tailwind CSS コンパイル 概要**

>Laravel Jetstream > Tailwind</br>
https://jetstream.laravel.com/2.x/concept-overview.html#tailwind

新しい Terminal window を開きます。</br>
laravel-jetstream をインストールしたディレクトリへ移動
```
cd /srv/www/laravel-jetstream
```
※ Tailwind CSS を編集中は ```npm run watch``` を実行します。</br>
watch コマンドで HTML （laravel の場合 .blade.php ）ファイルの変更を追いかけ、変更毎に CSS / JavaScript をコンパイルします。
```
npm run watch
```

応答（ **npm run watch は実行し続けるので control + C で終了します。**）
```
> watch
> mix watch

● Mix █████████████████████████ emitting (95%)
 emit

● Mix █████████████████████████ done (99%) plugins
 WebpackBar:done

✔ Mix
  Compiled successfully in 2.99s
   Laravel Mix v6.0.43

✔ Compiled Successfully in 2960ms
┌─────────────────────────────────────────────────────────────────────────────────────────┬──────────┐
│                                                                                    File │ Size     │
├─────────────────────────────────────────────────────────────────────────────────────────┼──────────┤
│                                                                              /js/app.js │ 694 KiB  │
│                                                                             css/app.css │ 47.5 KiB │
└─────────────────────────────────────────────────────────────────────────────────────────┴──────────┘
webpack compiled successfully
```
**npm run watch は実行し続けるので control + C で終了します。**

※ Tailwind CSS でファイルを編集中は、Terminal window で ```npm run watch``` を実行し続ける必要があります。そうしないと、Tailwind CSS での変更が反映されません。

## <a name="tailwindcss_compile_command"></a>**Tailwind CSS コンパイル コマンド**


開発のためにCSS / JavaScriptをコンパイルし、変更時に再コンパイル。※Tailwind CSS の編集中に実行します。</br>
Compile your CSS / JavaScript for development and recompile on change...
```
npm run watch
```

開発のためにCSS/JavaScriptをコンパイル</br>
Compile your CSS / JavaScript for development...
```
npm run dev
```

本番用にCSS/JavaScriptをコンパイル</br>
Compile your CSS / JavaScript for production...
```
npm run prod
```

## <a name="tailwindcss_browser_clear_cache"></a>**ブラウザのキャッシュを削除**

コンパイルし作成された app.css および app.js はブラウザにキャッシュされるため、ブラウザのキャッシュを消去および削除しておきます。</br>
※編集が反映されなくなったら改めてキャッシュを消去・削除してください。</br>
>Firefox のキャッシュを消去するには</br>
削除したいデータから、"キャッシュ" のみ選択すること</br>
https://support.mozilla.org/ja/kb/how-clear-firefox-cache

>Microsoft Edge の閲覧履歴を表示または削除する</br>
削除したいデータから、"キャッシュされた画像とファイル" のみ選択すること</br>
https://support.microsoft.com/ja-jp/microsoft-edge/microsoft-edge-%E3%81%AE%E9%96%B2%E8%A6%A7%E5%B1%A5%E6%AD%B4%E3%82%92%E8%A1%A8%E7%A4%BA%E3%81%BE%E3%81%9F%E3%81%AF%E5%89%8A%E9%99%A4%E3%81%99%E3%82%8B-00cf7943-a9e1-975a-a33d-ac10ce454ca4

## <a name="tailwindcss_cache_control_no_cache"></a>**開発中のWebSiteからローカルにキャッシュを保存させない**
作成中のWebSiteを他の人に確認してもらいたい場合、WebSite側からブラウザにキャッシュを残さない設定をしておきます。

*.blade.php の ```head``` 属性の中に ```Cache-Control``` を追加しておく。
```
<meta http-equiv="Cache-Control" content="no-cache">
```
記載例
```
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta http-equiv="Cache-Control" content="no-cache">
```
Laravel Jetstream に関係する .blade.php
```
/resources/views/layouts/app.blade.php
/resources/views/layouts/guest.blade.php
```

Laravel デフォルトの welcome.blade.php
```
/resources/views/welcome.blade.php
```

## <a name="tailwindcss_welcome_blade_php"></a>**welcome.blade.php で Tailwind CSS を使用する**
welcome.blade.php は、Laravel Jetstream のテンプレートコンテンツではないので ```css/app.css``` を読み込むように指定します。

welcome.blade.php の ```head``` 属性の中に ```<link rel="stylesheet" href="{{ mix('css/app.css') }}">``` を追加しておく。
```
    <link rel="stylesheet" href="{{ mix('css/app.css') }}">
</head>
```

## <a name="max_user_watches"></a>```max_user_watches``` **変更対象ファイルの監視上限数を変更する必要がある場合**

>Error: ENOSPC: System limit for number of file watchers reached<br>
https://blog.dksg.jp/2019/09/error-enospc-system-limit-for-number-of.html

現在の設定値を確認
```
cat /proc/sys/fs/inotify/max_user_watches
```

応答（ Amazon Linux 2 のデフォルト値 ）
```
8192
```

値を変更（ 524288 は、8192（デフォルト値）の64倍です ）
```
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***

>Get started with Tailwind CSS<br>
https://tailwindcss.com/docs/installation<br>
>Laravel Jetstream > Tailwind</br>
https://jetstream.laravel.com/2.x/concept-overview.html#tailwind</br>
>LError: ENOSPC: System limit for number of file watchers reached</br>
https://blog.dksg.jp/2019/09/error-enospc-system-limit-for-number-of.html