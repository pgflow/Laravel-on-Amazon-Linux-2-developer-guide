chmod 777 database && chmod 777 database/database.sqlite


sqlite を使用する場合、php artisan migrate のあと、


[@]$ php artisan migrate

   WARN  The SQLite database does not exist: database/database.sqlite.  

  Would you like to create it? (yes/no) [no]
❯ yes


SQLSTATE[HY000]: General error: 8 attempt to write a readonly database 
SQLSTATE[HY000]: General error: 14 unable to open database file 
権限を付与で解決できます。
sudo chown -R apache:ec2-user database/ database/database.sqlite
