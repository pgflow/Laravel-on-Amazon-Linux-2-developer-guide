# 補足資料：Laravel 9 Import Export Excel and CSV File Tutorial



+ [補足資料：Laravel 9 Import Export Excel and CSV File Tutorial](Laravel-9-Import-Export-Excel-and-CSV-File-Tutorial.md)


##


ディレクトリの権限を変更する
```
sudo chown -R ec2-user:ec2-user storage/
```

エクセルのコンポートをインストール
```
composer require psr/simple-cache:^1.0 maatwebsite/excel
```
応答
```
Info from https://repo.packagist.org: #StandWithUkraine
Using version ^3.1 for maatwebsite/excel
./composer.json has been updated
Running composer update psr/simple-cache maatwebsite/excel
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - maatwebsite/excel[3.1.36, ..., 3.1.x-dev] require phpoffice/phpspreadsheet ^1.18 -> satisfiable by phpoffice/phpspreadsheet[1.18.0, ..., 1.22.0].
    - maatwebsite/excel[3.1.0, ..., 3.1.25] require php ^7.0 -> your php version (8.0.16) does not satisfy that requirement.
    - maatwebsite/excel[3.1.26, ..., 3.1.35] require illuminate/support 5.8.*|^6.0|^7.0|^8.0 -> found illuminate/support[v5.8.0, ..., 5.8.x-dev, v6.0.0, ..., 6.x-dev, v7.0.0, ..., 7.x-dev, v8.0.0, ..., 8.x-dev] but these were not loaded, likely because it conflicts with another require.
    - phpoffice/phpspreadsheet[1.18.0, ..., 1.22.0] require ext-gd * -> it is missing from your system. Install or enable PHP's gd extension.
    - Root composer.json requires maatwebsite/excel ^3.1 -> satisfiable by maatwebsite/excel[3.1.0, ..., 3.1.x-dev].

To enable extensions, verify that they are enabled in your .ini files:
    - /etc/php.ini
    - /etc/php.d/20-bz2.ini
    - /etc/php.d/20-calendar.ini
    - /etc/php.d/20-ctype.ini
    - /etc/php.d/20-curl.ini
    - /etc/php.d/20-dom.ini
    - /etc/php.d/20-exif.ini
    - /etc/php.d/20-fileinfo.ini
    - /etc/php.d/20-ftp.ini
    - /etc/php.d/20-gettext.ini
    - /etc/php.d/20-iconv.ini
    - /etc/php.d/20-mbstring.ini
    - /etc/php.d/20-mysqlnd.ini
    - /etc/php.d/20-pdo.ini
    - /etc/php.d/20-phar.ini
    - /etc/php.d/20-simplexml.ini
    - /etc/php.d/20-sockets.ini
    - /etc/php.d/20-sqlite3.ini
    - /etc/php.d/20-tokenizer.ini
    - /etc/php.d/20-xml.ini
    - /etc/php.d/20-xmlwriter.ini
    - /etc/php.d/20-xsl.ini
    - /etc/php.d/20-zip.ini
    - /etc/php.d/30-mysqli.ini
    - /etc/php.d/30-pdo_mysql.ini
    - /etc/php.d/30-pdo_sqlite.ini
    - /etc/php.d/30-xmlreader.ini
You can also run `php --ini` in a terminal to see which files are used by PHP in CLI mode.
Alternatively, you can run Composer with `--ignore-platform-req=ext-gd` to temporarily ignore these required extensions.
You can also try re-running composer require with an explicit version constraint, e.g. "composer require maatwebsite/excel:*" to figure out if any version is installable, or "composer require maatwebsite/excel:^2.1" if you know which you need.

Installation failed, reverting ./composer.json and ./composer.lock to their original content.
```


再度実行（バージョンの指定は必ず必要）
```
composer require psr/simple-cache:^1.0 maatwebsite/excel
```

応答
```
./composer.json has been updated
Running composer update maatwebsite/excel
Loading composer repositories with package information
Updating dependencies
Nothing to modify in lock file
Installing dependencies from lock file (including require-dev)
Nothing to install, update or remove
Package phpoffice/phpexcel is abandoned, you should avoid using it. Use phpoffice/phpspreadsheet instead.
Generating optimized autoload files
Class PHPExcel_Properties located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Chart/Properties.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/bestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Exponential_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/exponentialBestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Linear_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/linearBestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Logarithmic_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/logarithmicBestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Polynomial_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/polynomialBestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
Class PHPExcel_Power_Best_Fit located in ./vendor/phpoffice/phpexcel/Classes/PHPExcel/Shared/trend/powerBestFitClass.php does not comply with psr-0 autoloading standard. Skipping.
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: jenssegers/agent
Discovered Package: laravel/fortify
Discovered Package: laravel/jetstream
Discovered Package: laravel/sail
Discovered Package: laravel/sanctum
Discovered Package: laravel/tinker
Discovered Package: livewire/livewire
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Discovered Package: spatie/laravel-ignition
Package manifest generated successfully.
80 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
```


Import クラス
```
php artisan make:import UsersImport --model=User
```
```
Import created successfully.
```

export クラス
```
php artisan make:export UsersExport --model=User
```
```
Export created successfully.
```

UserController コントローラー
```
php artisan make:controller UserController
```
```
Controller created successfully.
```