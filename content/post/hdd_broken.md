---
title: "HDD/SSDの復旧方法まとめ"
date: 2018-01-28T17:51:24+09:00
categories: [POST, Other]
thumbnail: "images/cat21449.jpg" 
toc: true 
---

# 事の始まり
SSDが壊れた．
またか，と思っていつもどおりddrescueの使い方を調べ始める．
去年は3台ほどHDDを壊し，ddrescueによって九死に一生を得てきた．ddrescueさえあれば大抵のことはどうにかなる．もう手慣れたものだ，そう，思っていた．このときまでは......


## 最初の悲劇
最初の悲劇はここで訪れる．そもそもデバイスを**認 識 し な い** ／(^o^)＼．
そんなんどうすればええねん・・・Ubuntuを何回か再起動するも，SATAを抜き差しするもさっぱりダメだった．

### systemrescueCD
システムを毎回すべて再起動するのは大変なので，小さいLinuxに復旧用ツールを詰め込んだ
systemrescueCDにメインを移すことにした．

あってよかった．

systemrescueCDに入れ替えた初回，なぜかデバイスを認識した！ktkr！！やs神


## ddrescueとは
次に出てくるのはご存知[ddrescue](https://ja.wikipedia.org/wiki/Ddrescue)．
端的に言えば，ddコマンドの復旧に特化したすごい版である．

使い方も非常にシンプル．

### 初動の使い方
~~~bash
Usage: ddrescue [options] infile outfile [logfile]

# ddrescue -v /dev/sdb /dev/sdc rescue.log
~~~
outfileはデバイスである必要はなく，ファイルにも書き出し可能である．
logfileは読み取れなかったセクタの再読み出しに使うので必ず指定する．

### 読み取れなかった箇所の読み取りを再度チャレンジ
```shell
# ddrescue -v -r 3 /dev/sdb /dev/sdc rescue.log
```
rは再読み取りを試みる回数である．

### ところが・・・
エラーサイズが小さくならない．いつもなら読み取れない領域なんてほんの少しで
何回か繰り返せば読めていたのに・・・．これがSSDの特性か．
r3で読み取り続けること15時間，ようやくエラーサイズが130MBになったものの
これ以上はもう物理的に無理だった．

## ディスク全体をクローン
いつもであれば，ディスクイメージを別のディスクへ書き込めば終了である．
とりえあえず，ディスクをマウントしてみる．

```
# mount windows.img /mnt
mount: wrong fs type, bad option, bad superblock on /dev/loop3,
       missing codepage or helper program, or other error

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
```
マウントできない？よく考えればディスク全体イメージなので，パーティションの先頭を指定してやる必要がある．

~~~bash
$ parted windows.img 
WARNING: You are not superuser.  Watch out for permissions.
GNU Parted 3.2
Using windows.img
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) unit b
(parted) p                                                                
Model:  (file)
Disk windows.img: 128035676160B
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start         End            Size          Type     File system  Flags
 1      4096B         28790751231B   28790747136B  primary  ntfs         boot
 2      28790751232B  98894348287B   70103597056B  primary  ntfs
 3      98894348288B  128035323903B  29140975616B  primary  ntfs         diag
~~~

~~~bash
$ sudo mount -o offset=4096 windows.img /mnt
$ ls /mnt
bootmgr  BOOTSECT.BAK  Recovery      $RECYCLE.BIN               Temp
BOOTNXT  found.000     Recovery.txt  System Volume Information  XGUFC
~~~
最初の領域は読めた！

```
$ sudo mount -o offset=28790751232 windows.img /mnt
ntfs_mst_post_read_fixup_warn: magic: 0x00000000  size: 4096   usa_ofs: 0  usa_count: 65535: Invalid argument
Index buffer (VCN 0x0) of directory inode 0x5 has a size (24) differing from the directory specified size (4096).
Failed to mount '/dev/loop3': Input/output error
NTFS is either inconsistent, or there is a hardware fault, or it's a
SoftRAID/FakeRAID hardware. In the first case run chkdsk /f on Windows
then reboot into Windows twice. The usage of the /f parameter is very
important! If the device is a SoftRAID/FakeRAID then first activate
it and mount a different device under the /dev/mapper/ directory, (e.g.
/dev/mapper/nvidia_eahaabcc1). Please see the 'dmraid' documentation
for more details.
```
読めない＼(^o^)／

~~~
$ sudo mount -o offset=98894348288 windows.img /mnt
$ ls /mnt
Recovery  Recovery.txt  System Volume Information
~~~
読めた！

このままだと，クローンしても起動できない．どうすんのこれ？

## パーティションテーブルが壊れているのでは？
ディスクが読めないのには大きく分けて2つがあると思う．

1. パーティションテーブルが読めない
 - testdiskを使う．可能ならパーティションテーブルを修復する．

1. ファイルシステムにエラーがある
 - NTFSの場合は，ntfsfixを使う．一般的にはfsckを使う．ntfsfixはfsck.ntfsとfsck.ntfs-3gのシンボリックリンク先らしい．

エラーの内容をggったところ，askubuntuに答えがあった．[NTFS drive corrupt](https://askubuntu.com/questions/825327/ntfs-drive-corrupt)

testdiskを使う．

## testdiskを使う
[testdisk](https://en.wikipedia.org/wiki/TestDisk)は主にデータ復旧やパーティションテーブルの復旧に使われる．

### 使い方
ディスクイメージそのものを指定する．
```
$ testdisk windows.img
```

~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org

  TestDisk is free software, and
  comes with ABSOLUTELY NO WARRANTY.

  Select a media (use Arrow keys, then press Enter):
  >Disk windows.img - 128 GB / 119 GiB

>[Proceed ]  [  Quit  ]

Note: Disk capacity must be correctly detected for a successful recovery.
If a disk listed above has incorrect size, check HD jumper settings, BIOS
detection, and install the latest OS patches and disk drivers.
~~~


~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org


Disk windows.img - 128 GB / 119 GiB

Please select the partition table type, press Enter when done.
>[Intel  ] Intel/PC partition
 [EFI GPT] EFI GPT partition map (Mac i386, some x86_64...)
 [Humax  ] Humax partition table
 [Mac    ] Apple partition map
 [None   ] Non partitioned media
 [Sun    ] Sun Solaris partition
 [XBox   ] XBox partition
 [Return ] Return to disk selection

Hint: None partition table type has been detected.
Note: Do NOT select 'None' for media with only a single partition. It's very
rare for a disk to be 'Non-partitioned'.
~~~
MBRを使っていれば，Intelを選ぶ．
2TiB未満はIntelを選ぶ．2TiB以上はGUIDパーティションテーブルを使っているのでEFI GPTを選ぶ，らしい．


~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org


Disk windows.img - 128 GB / 119 GiB
     CHS 15567 255 63 - sector size=512

     >[ Analyse  ] Analyse current partition structure and search for lost partitions
      [ Advanced ] Filesystem Utils
      [ Geometry ] Change disk geometry
      [ Options  ] Modify options
      [ MBR Code ] Write TestDisk MBR code to first sector
      [ Delete   ] Delete all data in the partition table
      [ Quit     ] Return to disk selection

Note: Correct disk geometry is required for a successful recovery. 'Analyse'
process may give some warnings if it thinks the logical geometry is mismatched.
~~~
~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org

Disk windows.img - 128 GB / 119 GiB - CHS 15567 255 63
Current partition structure:
     Partition                  Start        End    Size in sectors

      1 * HPFS - NTFS              0   0  9  3500  70 26   56231928
      2 P HPFS - NTFS           3500  70 27 12023  56  1  136921088
      3 P Windows RE(store)    12023  56  2 15566  19  5   56915968

*=Primary bootable  P=Primary  L=Logical  E=Extended  D=Deleted
>[Quick Search]  [ Backup ]
~~~
Quick Searchしてみる

~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org

Disk windows.img - 128 GB / 119 GiB - CHS 15567 255 63

The harddisk (128 GB / 119 GiB) seems too small! (< 168 GB / 157 GiB)
Check the harddisk size: HD jumpers settings, BIOS detection...

The following partition can't be recovered:
     Partition               Start        End    Size in sectors
     >  HPFS - NTFS          12023  56  1 20546  41 30  136921080

[ Continue ]
NTFS, blocksize=4096, 70 GB / 65 GiB
~~~
なんか見つかった．

~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org

Disk windows.img - 128 GB / 119 GiB - CHS 15567 255 63
     Partition               Start        End    Size in sectors
     >* HPFS - NTFS              0   0  9  3500  70 18   56231920
      D HPFS - NTFS           3500  70 27 12023  56  1  136921088
      D HPFS - NTFS           3500  70 35 12023  56  1  136921080
      P HPFS - NTFS          12023  56  2 15566  19  5   56915968

Structure: Ok.  Use Up/Down Arrow keys to select partition.
Use Left/Right Arrow keys to CHANGE partition characteristics:
*=Primary bootable  P=Primary  L=Logical  E=Extended  D=Deleted
Keys A: add partition, L: load backup, T: change type, P: list files,
Enter: to continue
~~~
なんか増えた．testdiskはここで正しいパーティションテーブルの候補を提示してくる．
現段階ではメインの領域候補が両方ともDeletedになっていてこのまま書き込むと削除されてしまう．
正しいテーブルはどちらなのか，P: list filesとなっているので調べてみる．

#### 1個目
~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org
     HPFS - NTFS           3500  70 27 12023  56  1  136921088
     Directory /

     >dr-xr-xr-x     0     0         0  7-Jul-2017 09:05 .
      dr-xr-xr-x     0     0         0  7-Jul-2017 09:05 ..
      -r--r--r--     0     0 4294967296 25-Jul-2017 17:15 pagefile.sys

~~~

#### 2個目
~~~
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org

     HPFS - NTFS           3500  70 35 12023  56  1  136921080

     Can't open filesystem. Filesystem seems damaged.
~~~

こんなん無理ゲーやん・・・(´・ω・｀)

##### なんかこうDeeper Searchとかやってみたけど無理だった


## 意味はないだろうけど，ntfsfixもやってみた
ntfsfixはディスクイメージに対応しないので，[loopバックデバイス](https://qiita.com/mokrai/items/55e
0061e6d5fca78631f)にマウントしてやってみた．

[Running fsck on filesystems inside a partitioned image](https://chrisdown.name/2011/06/01/fsck-partitions-inside-an-image.html)
~~~
$ losetup -o 28790751232 /dev/loop0 windows.img # 先ほどの値を使う
$ sudo ntfsfix /dev/loop0
~~~

~~~
$ sudo ntfsfix  /dev/loop0
Mounting volume... ntfs_mst_post_read_fixup_warn: magic: 0x00000000  size: 4096   usa_ofs: 0  usa_count: 65535: Invalid argument
Index buffer (VCN 0x0) of directory inode 0x5 has a size (24) differing from the directory specified size (4096).
FAILED
Attempting to correct errors... 
Processing $MFT and $MFTMirr...
Reading $MFT... OK
Reading $MFTMirr... OK
Comparing $MFTMirr to $MFT... OK
Processing of $MFT and $MFTMirr completed successfully.
Setting required flags on partition... OK
Going to empty the journal ($LogFile)... OK
ntfs_mst_post_read_fixup_warn: magic: 0x00000000  size: 4096   usa_ofs: 0  usa_count: 65535: Invalid argument
Index buffer (VCN 0x0) of directory inode 0x5 has a size (24) differing from the directory specified size (4096).
Remount failed: Input/output error
~~~

むーりぃー
無理すぎィ！


## 万策尽きた／(^o^)＼
ddrescueには可視化ツールとしてddrescueviewというのがあるらしい．
最後に一応確認してみる．
ddrescueで取得したログファイルを使う．
~~~shell
$ ddrescueview ddrescue.log
~~~

{{% img src="images/ddrescueview.png" w="100%" h="100%" %}}

いや，無理でしょ，これ......


## 運良く復旧出来たら
~~~bash
$ dd if=windows.img of=/dev/sdx bs=512k
~~~
