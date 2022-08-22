# Command Line Reference


## 目次
+ [Linux](#linux)
+ [Tailwind CSS](#tailwind_css)
+ [MariaDB](#mariadb)
+ [Git](#git)

***

## <a name="linux"></a>Linux
Amazon Linux 2 インスタンスのメモリ空き容量を（2秒毎に更新）確認できます。<br>```control + C``` を押して終了出来ます。
```
watch -d free -m
```

## <a name="tailwind_css"></a>Tailwind CSS
Tailwind CSS の編集中に実行します。
<br>```control + C``` を押して終了出来ます。
```
npm run watch
```

## <a name="mariadb"></a>MariaDB

### MariaDB ログイン
```
mariadb -u root -p
```
応答（パスワードを入力）
```
Enter password:
```

### ユーザー権限の許可
対象ユーザーにすべての権限を許可
```
grant all on データベース名.デーブル名 to 'ユーザー名';
```

対象ユーザーに、`select,delete,insert,update` の権限を許可
```
grant select,delete,insert,update on データベース名.デーブル名 to 'ユーザー名';
```

必ず権限テーブルを flush して、設定を反映する。
```
flush privileges;
```

設定を確認します。
```
show grants for 'ユーザー名';
```

```
+------------------------------------------------------------------------------------------------------+
| Grants for ユーザ名@%                                                                                   |
+------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `ユーザ名`@`%` IDENTIFIED BY PASSWORD '*password' |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `データベース`.`テーブル` TO `ユーザ名`@`%`      |
+------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)
```

### ユーザー権限の取り消し
対象ユーザーから、`drop` の権限を取り消し
```
revoke drop on データベース名.デーブル名 from 'ユーザー名';
```

必ず権限テーブルを flush して、設定を反映する。
```
flush privileges;
```

## <a name="git"></a>Git

### 削除しても表示されるリモートブランチの消去

origin （origin 以外の名前なら作成した名前）を確認する
```
git remote show origin
```

応答（例）
```
~
  Remote branches:
    feature stale
    main    tracked
~
```

 Remote branches: が `tracked` `（追跡）` であれば、branch は存在し利用中です。

 Remote branches: が `stale` `（古い）` であれば、削除済み branch です。 この削除済み branch が VScode に表示されることがある。ため消去します。

消去するには ```prune``` を実行。
```
git remote prune origin
```
消去される branch を事前に確認したい場合は ```dry-run``` 使用する。
```
git remote prune origin --dry-run
```

削除が行われたか、origin （origin 以外の名前なら作成した名前）を確認する
```
git remote show origin
```

応答（例）
```
~
  Remote branches:
    main    tracked
~
```
