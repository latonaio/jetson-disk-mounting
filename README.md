# jetson-disk-mounting #
本手順はJetsonにディスク（SSD、SDカード等）を新規に搭載し、それをマウントするまでの手順を示しています。  
この手順内ではNVME SSDを使用し、マウント先は/var/lib/dockerとしています。

 
### **【手順】** 
Jetsonにディスクを搭載します。（省略）  
当該ディスクをフォーマットします。  
マウント元のディレクトリのデータを、マウント先にコピーします。  
fstabに登録し、起動時に自動的にマウントするよう設定します。  


### **【手順詳細】**  
**0.ディスク認識の確認**  
マウントしたディスクがOSから認識できるか下記コマンドから確認します。  

```
$ sudo fdisk -l
```

実行結果例  
```
Disk /dev/nvme0n1: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: B272CBBD-55BC-4083-B62D-15B750727EBD

```
 
ここでは、nvme0n1 が今、パーティションを作成したいディスクです。ここは適宜、変更してください。  

**1.パーティションの作成**  
fdiskコマンドで、インタラクティブにサイズ等の設定を入力し、パーティションを作成します。 

```
$ sudo fdisk /dev/nvme0n1

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
```

```
Command (m for help): n
Partition number (1-128, default 1):
First sector (34-500118158, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-500118158, default 500118158): 250118158　 <=Defaultではディスク容量を最大まで使うので、複数ディレクトリにマウントしたい場合は適当な値を入力する。

Created a new partition 1 of type 'Linux filesystem' and of size 119.3 GiB.
Partition #1 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): w
```

新規のパーティションが作成されていることを確認します。  
```
$ sudo fdisk -l
Disk /dev/nvme0n1: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 1DA75FA1-1F2B-482E-B498-5D410725C5D0
```

```
Device         Start       End   Sectors   Size Type
/dev/nvme0n1p1  2048 250118158 250116111 119.3G Linux filesystem　<=パーティションが作成されていることを確認 
```
**2.ディスクのフォーマット**  
```
$ sudo mkfs.ext4 /dev/nvme0n1p1
```

**3.データのマウント先へのコピー**  
```
# コピー先となる仮ディレクトリを作成する。
sudo mkdir /mnt/data
# 作成したパーティションを作成したディレクトリにマウントする。
sudo mount /dev/nvme0n1p1 /mnt/data/
# マウント元のデータをコピーする。
sudo rsync -aXS /var/lib/docker/. /mnt/data/.
# ストレージのマウントを外す。
sudo umount /dev/nvme0n1p1
```
**4.fstabへの登録と、起動時自動マウントの設定**  
```
# fstabのファイルを開く
sudo vi /etc/fstab
# 下記の設定をファイルの末尾に追加する。
/dev/nvme0n1p1 　/var/lib/docker  ext4  defaults  0 1
# 再起動する
sudo reboot
```
