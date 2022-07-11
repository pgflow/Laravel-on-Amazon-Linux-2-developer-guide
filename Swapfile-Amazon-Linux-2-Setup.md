# <a name="Swapfile-Amazon-Linux-2-Setup"></a>Swapfile Amazon Linux 2 Setup

作成日：2022/06/27<br>

このドキュメントは、Amazon Linux 2 に Swapfile の設定を行う事を目的に書かれています。作成時より時が経つと書いている手順通りに出来なくなる可能性があります。

## 前提

+ [Amazon Lightsail](https://lightsail.aws.amazon.com/)
で Amazon Linux 2 インスタンスを新規作成済みであること。<br>

+ Windows Terminal などで、SSH 接続出来る準備が出来ていること。


## 目次
+ [インスタンスのメモリ空き容量を確認する](#watch_free_m)
+ [スワップファイルの作成と開始](#dd_swapfile)
+ [スワップファイルの停止と削除](#rm_swapfile)
***

## <a name="watch_free_m"></a>インスタンスのメモリ空き容量を確認する
Amazon Linux 2 インスタンスのメモリ空き容量を（2秒毎に更新）確認できます。
`control` + `c` を押して `watch -d free -m` を終了出来ます。

```
watch -d free -m
```
応答（インスタンスサイズ：1 GB RAM、1 vCPU、40 GB の SSD の場合）
```
Every 2.0s: free -m                                                           Mon Jun 27 04:21:26 2022

              total        used        free      shared  buff/cache   available
Mem:            982         221         438           0         322         614
Swap:             0           0           0
```

`control` + `c` で `watch -d free -m` を終了。

## <a name="dd_swapfile"></a>スワップファイルを作成
>スワップファイルを使用して、Amazon EC2 インスタンスのスワップ領域として機能するようにメモリを割り当てるにはどうすればよいですか?<br>
>https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-memory-swap-file/

以下の dd コマンドで、swapfile 4 GB (128 MB x 32) を作成します。※契約インスタンスのメモリ（システムRAM）の4倍以内で調整するとよい。（実際に動かしているアプリケーションによります）<br>
command からの返答に少し時間（30秒程度）が、かかります。
```
sudo dd if=/dev/zero of=/swapfile bs=128M count=32
```

応答
```
32+0 records in
32+0 records out
4294967296 bytes (4.3 GB) copied, 64.8561 s, 66.2 MB/s
```

swapfile の読み書きのアクセス許可を更新します。
```
sudo chmod 600 /swapfile
```

swapfile 領域のセットアップ
```
sudo mkswap /swapfile
```

応答
```
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=46129958-e718-4d78-b22a-4eab04f92ffb
```

スワップ領域に swapfile を追加して、swapfile を即座に使用できるようにします。
```
sudo swapon /swapfile
```

正常に完了したことを確認します。
```
sudo swapon -s
```

応答
```
Filename                                Type            Size    Used    Priority
/swapfile                               file            2097148 0       -2
```

/etc/fstab ファイルを編集して、起動時に swapfile を起動します。
```
sudo nano /etc/fstab
```

ファイルの末尾に次の新しい行を追加し、ファイルを保存して終了します。<br>
`control + o` で保存。`control + x` で終了。
```
/swapfile swap swap defaults 0 0
```

空きメモリの様子を `watch -d free -m` で確認します。
```
watch -d free -m
```
応答（Swap total が 2047 になりました。）
```
Every 2.0s: free -m                                                           Mon Jun 27 04:42:16 2022

              total        used        free      shared  buff/cache   available
Mem:            982         154         171           0         656         684
Swap:          2047           0        2047
```

## <a name="rm_swapfile"></a>スワップファイルの停止と削除

>15.2.3. スワップファイルの削除<br>
>https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/storage_administration_guide/swap-removing-file

swapfile 領域の swapfile を `swapoff` して、swapfile の使用をやめる。
```
sudo swapoff /swapfile
```

`watch -d free -m` で、Swap: の状態を確認する。
```
watch -d free -m
```

応答（Swap: total が 0 になっているか確認 ）
```
Every 2.0s: free -m                                                           Mon Jun 27 04:46:31 2022

              total        used        free      shared  buff/cache   available
Mem:            982         152         170           0         659         686
Swap:             0           0           0
```

swapfile を無効化する
```
sudo swapoff -v /swapfile
```

応答
```
swapoff /swapfile
swapoff: /swapfile: swapoff failed: Invalid argument
```

/etc/fstab ファイルを編集して、起動時に swapfile を起動するエントリを削除する。
```
sudo nano /etc/fstab
```

ファイルの末尾に追加した以下の行を削除し、ファイルを保存して終了します。<br>
`control + o` で保存。`control + x` で終了。
```
/swapfile swap swap defaults 0 0
```

実際の swapfile を削除します。 
```
sudo rm /swapfile
```

***
+ [pageTop](#pageTop)
+ [README](README.md)
***
