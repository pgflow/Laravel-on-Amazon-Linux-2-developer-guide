# Visual Studio Code on Amazon Linux 2 install guide



+ [Remote - SSH 機能拡張 を使用して SSH 接続できないエラー](#remote_ssh_error)
    + [Bad configuration option: \357\273\277](#remote_ssh_\357\273\277)
    + [Permissions 0644 for 'xxxxxxxxxxx.pem' are too open.](#remote_ssh_pem_permissions)

***
## <a name="#remote_ssh_error"></a>Remote - SSH 機能拡張 を使用して SSH 接続できないエラー

## <a name="remote_ssh_\357\273\277"></a>> Bad configuration option: \357\273\277
※ Windows 環境だと関係ないかもしれません。

**config ファイルの確認と変更ポイント**

ローカルコンピュータの ユーザー名/.ssh/ ディレクトリ内の、<br>
"config" ファイル<br>
エンコードタイプが "UTF-8 with BOM" で<br>
改行コードが "CRLF" の場合に以下のエラーが現れる場合がある。<br>
```
Bad configuration option: \357\273\277
```

"config" ファイルの<br>
エンコードタイプを "UTF-8" に変更し<br>
改行コードを "LF" に変更して保存し直すと解決する。

## <a name="remote_ssh_pem_permissions"></a>> Permissions 0644 for 'xxxxxxxxxxx.pem' are too open.
※ Windows 環境だと関係ないかもしれません。<br>
以下の エラーが表示される場合があります。
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: UNPROTECTED PRIVATE KEY FILE! @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'xxxxxxxxxxx.pem' are too open.
```

ローカルコンピュータの ユーザー名/.ssh/ ディレクトリ内の、<br>
"xxxxxxxxxxx.pem" ファイルのパーミッション を変更します。
```
chmod 600 xxxxxxxxxxx.pem
```