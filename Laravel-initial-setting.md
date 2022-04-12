# Laravel initial setting
+ [Laravel 初期設定 日本時間と日本語環境へ変更](#laravel_timezone)
    + [タイムゾーン](#laravel_timezone)
    + [ローカルコンフィギュレーション](#laravel_locale_configuration)
    + [ローカルコンフィギュレーション関連ファイルの作成](#laravel_locale_configuration_file)
    + [Laravel Jetstream ユーザープロファイル 日本語化](#laravel_jetstream_profile_ja)
***
## Laravel 初期設定 日本時間と日本語環境へ変更

>Laravel 9 ローカリゼーション</br>
>https://laravel.com/docs/9.x/localization</br>
>Locale ja_JP Faker 13.3.4 documentation</br>
>https://faker.readthedocs.io/en/master/locales/ja_JP.html

## <a name='laravel_timezone'></a>タイムゾーン

編集対象ファイル
```
/config/app.php
```

編集箇所とポイント
```
/*
|--------------------------------------------------------------------------
| Application Timezone
|--------------------------------------------------------------------------
|
| Here you may specify the default timezone for your application, which
| will be used by the PHP date and date-time functions. We have gone
| ahead and set this to a sensible default for you out of the box.
|
*/

'timezone' => 'UTC',
```
デフォルトのタイムゾーンは、'UTC'（協定世界時）
```
'timezone' => 'UTC',
```

日本時間へ変更
```
'timezone' => 'Asia/Tokyo',
```

## <a name='laravel_locale_configuration'></a>ローカルコンフィギュレーション

編集対象ファイル
```
/config/app.php
```

編集箇所とポイント
```
/*
|--------------------------------------------------------------------------
| Application Locale Configuration
|--------------------------------------------------------------------------
|
| The application locale determines the default locale that will be used
| by the translation service provider. You are free to set this value
| to any of the locales which will be supported by the application.
|
*/

'locale' => 'en',

/*
|--------------------------------------------------------------------------
| Application Fallback Locale
|--------------------------------------------------------------------------
|
| The fallback locale determines the locale to use when the current one
| is not available. You may change the value to correspond to any of
| the language folders that are provided through your application.
|
*/

'fallback_locale' => 'en',

/*
|--------------------------------------------------------------------------
| Faker Locale
|--------------------------------------------------------------------------
|
| This locale will be used by the Faker PHP library when generating fake
| data for your database seeds. For example, this will be used to get
| localized telephone numbers, street address information and more.
|
*/

'faker_locale' => 'en_US',
```

デフォルトのロケールは、'en'（英語）
```
'locale' => 'en',
```
日本語に変更する、'ja'（日本語）
```
'locale' => 'ja',
```

デフォルトのフォールバックロケールは、'en'（英語）
```
'fallback_locale' => 'en',
```
日本語に変更する、'ja'（日本語）
```
'fallback_locale' => 'ja',
```

デフォルトのフェイカーロケール、'en_US'（英語-アメリカ）
```
'faker_locale' => 'en_US',
```
日本語に変更する、'ja-JP'（日本語-日本）
```
'faker_locale' => 'ja-JP',
```

## <a name='laravel_locale_configuration_file'></a>ローカルコンフィギュレーション関連ファイルの作成

'en'（英語）のデフォルトファイル（ 例：Laravel Jetstream ）を確認します。
```
/lang/en.json
/lang/en/auth.php
/lang/en/pagination.php
/lang/en/passwords.php
/lang/en/validation.php
```

'en'（英語）をコピーして'jp'（日本語）版のファイルを作成します。
```
/lang/jp.json
/lang/jp/auth.php
/lang/jp/pagination.php
/lang/jp/passwords.php
/lang/jp/validation.php
```

/lang/jp/auth.php を日本語化してみます。

編集前の英語のエラーメッセージ
```
'failed' => 'These credentials do not match our records.',
'password' => 'The provided password is incorrect.',
'throttle' => 'Too many login attempts. Please try again in :seconds seconds.',
```

編集語の日本語のエラーメッセージ
```
'failed' => 'これらの資格情報は私たちの記録と一致しません。',
'password' => '提供されたパスワードが正しくありません。',
'throttle' => 'ログイン試行回数が多すぎます。 ：秒秒で再試行してください。',
```

以下のファイルを日本語化すると全てのメッセージが日本語になります。
```
/lang/jp/auth.php
/lang/jp/pagination.php
/lang/jp/passwords.php
/lang/jp/validation.php
```

## <a name='laravel_jetstream_profile_ja'></a>Laravel Jetstream ユーザープロファイル 日本語化

例として、ユーザープロファイルページの日本語化を説明</br>
http://example.xxx/user/profile

profile ページに関連する blade テンプレートは以下の通り。
```
/resources/views/profile/delete-user-form.blade.php
/resources/views/profile/logout-other-browser-sessions-form.blade.php
/resources/views/profile/show.blade.php
/resources/views/profile/two-factor-authentication-form.blade.php
/resources/views/profile/update-password-form.blade.php
/resources/views/profile/update-profile-information-form.blade.php
```

メッセージは以下の方法で表示されている。
__('メッセージ') の形式で、'jp.json' に書かれた、'Profile Information' 書かれたメッセージを読み込みます。'jp.json' になにも書いていなければ、 __('メッセージ') の 'メッセージ' 部分をそのまま表示します。

/resources/views/profile/update-profile-information-form.blade.php のメッセージを日本語化してみます。</br>
blade テンプレートから以下の形式のメッセージを探します。</br>
{{ __(' ') }}
```
{{ __('Profile Information') }}
```

/lang/jp.json に以下のメッセージを書き足します。</br>
"message": "メッセージ"
```
"Profile Information": "プロファイル情報"
```

メッセージは、' , ' を付けて増やすことができます。
最後に追加したメッセージの末尾に ' , ' を付けないでください。
```
"Profile Information": "プロファイル情報",
"Update your account's profile information and email address.": "アカウントのプロファイル情報とメールアドレスを更新します。",
"Select A New Photo": "新しい写真を選択",
"Remove Photo": "写真を削除",
"Saved.": "保存しました。",
"Save": "保存する"
```

コメントを書きたい場合
json はコメントを書く方法がないため、以下のようにメッセージをコメント代わりにします。
```
"#": "コメント",
```
***
+ [pageTop](#pageTop)
+ [README](README.md)
***
