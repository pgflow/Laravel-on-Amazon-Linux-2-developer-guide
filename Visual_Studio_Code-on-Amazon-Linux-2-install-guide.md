エラーとその対処

## Visual_Studio_Code-on-Amazon-Linux-2-install-guide.md


Visual Studio Code からSSH 接続出来ない
SSH 接続出来ない

Remote - SSH 機能拡張

+ [Remote - SSH 機能拡張 を使用して SSH 接続できないエラー](#remote_ssh_\357\273\277)
## <a name="remote_ssh_\357\273\277"></a>Remote - SSH 機能拡張 を使用して SSH 接続できないエラー

## Bad configuration option: \357\273\277
※ Windows 環境だと関係ないかもしれません。

**config ファイルの確認と変更ポイント**

ローカルコンピュータの ユーザー名/.ssh/ ディレクトリ内の、<br>
"config" ファイル<br>
エンコードタイプが "UTF-8 with BOM" で<br>
改行コードが "CRLF" の場合に<br>
"Bad configuration option: \357\273\277" エラーが現れる場合がある。

"config" ファイルの<br>
エンコードタイプを "UTF-8" に変更し<br>
改行コードを "LF" に変更して保存し直すと解決する。