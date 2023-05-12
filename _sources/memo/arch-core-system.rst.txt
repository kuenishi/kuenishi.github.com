#####################################
Arch Linux Installation & Maintenance
#####################################

2017/10/19 - 2017/10/20

2018/9/9 Mac mini mid 2011 のインストールを追加
2018/3/15 カーネルパッチの部分を削除
2018/4/6 Trouble Shooting追加
2020/6/17 btrfs まわりを追加

ISO Image Download
------------------

https://www.archlinux.org/download/ からPGP署名ファイルとISOイメージをダウンロード::

  $ wget http://ftp.jaist.ac.jp/pub/Linux/ArchLinux/iso/2017.10.01/archlinux-2017.10.01-x86_64.iso
  $ wget https://www.archlinux.org/iso/2017.10.01/archlinux-2017.10.01-x86_64.iso.sig
  $ ls archlinux*
  archlinux-2017.10.01-x86_64.iso  archlinux-2017.10.01-x86_64.iso.sig

ダウンロードしたファイルの検証。公開鍵がないので一旦失敗する::

  $ LANG=C gpg --verbose --verify archlinux-2017.10.01-x86_64.iso.sig
  gpg: assuming signed data in 'archlinux-2017.10.01-x86_64.iso'
  gpg: Signature made Sun Oct  1 14:29:43 2017 JST
  gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
  gpg: Can't check signature: No public key


公開鍵を調べてとりあえずインポートする::

  $ gpg --search-key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
  gpg: data source: https://176.9.147.41:443
  (1)     Pierre Schmitz <pierre@archlinux.de>
          2048 bit RSA key 7F2D434B9741E8AC, 作成: 2011-04-10
  Keys 1-1 of 1 for "4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC".  番号(s)、N)次、またはQ)中止を入力してください >N
  $ gpg --recv-keys  7F2D434B9741E8AC
  gpg: 鍵7F2D434B9741E8AC: 公開鍵"Pierre Schmitz <pierre@archlinux.de>"をインポートしました
  gpg: marginals needed: 3  completes needed: 1  trust model: pgp
  gpg: 深さ: 0  有効性:   2  署名:   1  信用: 0-, 0q, 0n, 0m, 0f, 2u
  gpg: 深さ: 1  有効性:   1  署名:   0  信用: 0-, 0q, 0n, 0m, 1f, 0u
  gpg: 次回の信用データベース検査は、2018-07-11です
  gpg:           処理数の合計: 1
  gpg:             インポート: 1

改めてVerify::

  $ LANG=C gpg --verify archlinux-2017.10.01-x86_64.iso.sig
  gpg: assuming signed data in 'archlinux-2017.10.01-x86_64.iso'
  gpg: Signature made Sun Oct  1 14:29:43 2017 JST
  gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
  gpg: Good signature from "Pierre Schmitz <pierre@archlinux.de>" [unknown]
  gpg: WARNING: This key is not certified with a trusted signature!
  gpg:          There is no indication that the signature belongs to the owner.
  Primary key fingerprint: 4AA4 767B BC9C 4B1D 18AE  28B7 7F2D 434B 9741 E8AC

`イメージをUSBメモリに焼く <https://wiki.archlinux.jp/index.php/USB_%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%83%A1%E3%83%87%E3%82%A3%E3%82%A2>`_ ::

   $ sudo dmesg | tail
   [ 6587.846736] scsi 8:0:0:0: Direct-Access     Ut165    USB2FlashStorage 0.00 PQ: 0 ANSI: 2
   [ 6587.847379] sd 8:0:0:0: Attached scsi generic sg6 type 0
   [ 6587.848372] sd 8:0:0:0: [sdf] 7897040 512-byte logical blocks: (4.04 GB/3.76 GiB)
   [ 6587.849183] sd 8:0:0:0: [sdf] Write Protect is off
   [ 6587.849185] sd 8:0:0:0: [sdf] Mode Sense: 00 00 00 00
   [ 6587.850081] sd 8:0:0:0: [sdf] Asking for cache data failed
   [ 6587.850087] sd 8:0:0:0: [sdf] Assuming drive cache: write through
   [ 6588.007742]  sdf:
   [ 6588.400298] sd 8:0:0:0: [sdf] Attached SCSI removable disk
   [ 7272.546160]  sdf: sdf1 sdf2
   $ sudo dd if=archlinux-2017.10.01-x86_64.iso of=/dev/sdf status=progress && sudo sync
   544993792 bytes (545 MB, 520 MiB) copied, 189 s, 2.9 MB/s
   1071104+0 レコード入力
   1071104+0 レコード出力
   548405248 bytes (548 MB, 523 MiB) copied, 201.131 s, 2.7 MB/s


Installation
------------

ThinkPad届く。

Secure Bootをオフに必ずしておく。
Bootパスはまあこの際後でもいっか。

Windows Proのライセンスキーがパッケージや本体のどこにも書いてないので、
Windowsを起動して一旦プロダクトキーを抜いておく（おそらくThinkPadのCPU
IDかシリアルナンバーか何かをみて認証していると思われる。ブート後::

  root@archiso ~ # passwd
  root@archiso ~ # systemctl start sshd
  root@archiso ~ # ip addr

以後SSHで入って作業（優先で接続するために予めRJ45アダプタを買っておく）。ディスク名を確認::

  # fdisk -l
  Disk /dev/nvme0n1: 477 GiB, 512110190592 bytes, 1000215216 sectors
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disklabel type: gpt
  Disk identifier: 25A27C49-25BC-4715-8A09-A722ABE9260D

  Device             Start        End   Sectors   Size Type
  /dev/nvme0n1p1      2048     534527    532480   260M EFI System
  /dev/nvme0n1p2    534528     567295     32768    16M Microsoft reserved
  /dev/nvme0n1p3    567296  998166527 997599232 475.7G Microsoft basic data
  /dev/nvme0n1p4 998166528 1000214527   2048000  1000M Windows recovery environment

パーティションを切る::

  root@archiso ~ # parted /dev/nvme0n1 -s mklabel gpt -s mkpart ESP fat32 1MiB 513MiB -s set 1 boot on -s mkpart primary ext4 513MiB 100%
  root@archiso ~ # fdisk -l
  Disk /dev/nvme0n1: 477 GiB, 512110190592 bytes, 1000215216 sectors
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disklabel type: gpt
  Disk identifier: 1E26B015-ABC4-4949-8410-7F43A11ECE11

  Device           Start        End   Sectors   Size Type
  /dev/nvme0n1p1    2048    1050623   1048576   512M EFI System
  /dev/nvme0n1p2 1050624 1000214527 999163904 476.4G Linux filesystem

ブートセクタをフォーマットする::

  root@archiso ~ # mkfs.vfat -F32 /dev/nvme0n1p1
  mkfs.fat 4.1 (2017-01-24)
  root@archiso ~ # parted /dev/nvme0n1 print
  Model: Unknown (unknown)
  Disk /dev/nvme0n1: 512GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End    Size   File system  Name  Flags
  1      1049kB  538MB  537MB  fat32              boot, esp
  2      538MB   512GB  512GB

ルートパーティションを暗号化する。このとき入れるパスワードが、マシン起
動のときに毎回きかれるパスワードになる。このパスワードを覚えておけばあ
とで追加もできる。というか、覚えられるパスワードにしておこう。::

  root@archiso ~ # cryptsetup luksFormat /dev/nvme0n1p2

  WARNING!
  ========
  This will overwrite data on /dev/nvme0n1p2 irrevocably.

  Are you sure? (Type uppercase yes): YES
  Enter passphrase:
  Verify passphrase:

LUKSはかなり高機能で、YubiKeyで鍵を渡して起動とか、USBメモリ上の特定の
パスから拾ってくるとかいろいろできるらしい。initrdの中にパスワードファ
イルを仕込んでおくこともできるが、ここは素直に覚えやすいものを設定して
おくことにした。暗号化したやつをDevice Mapperで見えるようにする。ここ
では ``crypt-root`` と適当に名前をつける。それを ext4 でフォーマットま
でしとく。::

  root@archiso ~ # cryptsetup open /dev/nvme0n1p2 crypt-root
  Enter passphrase for /dev/nvme0n1p2:
  root@archiso ~ # mkfs.ext4 /dev/mapper/crypt-root
  mke2fs 1.43.6 (29-Aug-2017)
  Creating filesystem with 124894976 4k blocks and 31227904 inodes
  Filesystem UUID: 01dbc4c4-bebe-4ee5-92a1-2568aaef4c1f
  Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000

        Allocating group tables: done
        Writing inode tables: done
        Creating journal (262144 blocks): done
        Writing superblocks and filesystem accounting information: done

フォーマットしたものをと、ブートパーティションをマウントする。::

  root@archiso ~ # mount /dev/mapper/crypt-root /mnt
  root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot
  mount: /mnt/boot: mount point does not exist.
  32 root@archiso ~ # mkdir /mnt/boot
  root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot
  root@archiso ~ # timedatectl set-ntp true
  root@archiso ~ # timedatectl status
        Local time: Thu 2017-10-19 03:17:12 UTC
    Universal time: Thu 2017-10-19 03:17:12 UTC
          RTC time: Thu 2017-10-19 03:17:12
         Time zone: UTC (UTC, +0000)
   Network time on: yes
  NTP synchronized: yes
   RTC in local TZ: no

ミラーを日本だけのものにしとく。grepだと '--' という余計なものが入るの
で、消しておくこと。ベースシステムを ``pacstrap`` でインストールしてか
ら、この時点で ``/etc/fstab`` を作っておく。::

  # grep Japan -A 1 /etc/pacman.d/mirrorlist > mirrorlist
  (remove  -- here)
  # vi mirrorlist
  # mv mirrorlist /etc/pacman.d/
  # pacstrap /mnt base base-devel linux linux-firmware
  # genfstab -U /mnt >> /mnt/etc/fstab

 ``chroot`` して、OS環境の作成を開始する。まず、基本的なファイルを先においておく。::

  root@archiso ~ # arch-chroot /mnt
  [root@archiso /]# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
  [root@archiso /]# echo "LANG=en_US.UTF-8" > /etc/locale.conf
  [root@archiso /]# locale-gen
  [root@archiso /]# echo KEYMAP=us > /etc/vconsole.conf
  [root@archiso /]# echo utaha > /etc/hostname
  [root@archiso /]# pacman -S emacs-nox
  [root@archiso /]# emacs /etc/hosts
  (set hostname.localdomain)

ひととおり設定できたら、initramfsを作る。 ``/etc/mkinitcpio.conf`` を
編集して、 ``systemd`` と ``sd-encrypt`` を `HOOKSに追加する
<https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#mkinitcpio>`_
。いまのところはこういう並びでうまくいっている。 ``sd-encrypt`` は ``block`` の前にないとだめそう。::

  HOOKS="base systemd udev autodetect modconf sd-encrypt block filesystems keyboard fsck"

巷には LVM と ``encrypt`` を使ってなぜか ``systemd-boot`` でうまく行っ
ている例があるみたいだが、少なくとも手元では成功しなかったので鵜呑みに
するのは危険だろう。 `If you use the systemd hook then you have to use
sd-encrypt and
sd-lvm. <https://bbs.archlinux.org/viewtopic.php?pid=1673320#p1673320>`_
とも書いてあるし。これが編集できたら initrd を作成。::

  # mkinitcpio -p linux

次はブートセクタの作成。まずUUIDを確認して、とりあえずもろもろのファイルを作っておく。::

  # cryptsetup luksUUID /dev/nvme0n1p2
  1cf54a61-cd17-43e3-ad00-bf94c29dc922
  # bootctl --path=/boot install
  # pacman -S intel-ucode

``intel-ucode`` は `CPUのマイクロコードをアップデートしてくれるものらしい <https://wiki.archlinux.jp/index.php/%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%82%B3%E3%83%BC%E3%83%89#Intel_.E3.81.AE.E3.83.9E.E3.82.A4.E3.82.AF.E3.83.AD.E3.82.B3.E3.83.BC.E3.83.89.E3.81.AE.E3.82.A2.E3.83.83.E3.83.97.E3.83.87.E3.83.BC.E3.83.88.E3.82.92.E6.9C.89.E5.8A.B9.E3.81.AB.E3.81.99.E3.82.8B>`_ 。

次にブートローダの設定を作成。::

  # emacs /boot/loader/loader.conf
  # cat /boot/loader/loader.conf
  timeout 5
  ## default 64783ef8e33e4205862d8f26c79b569c-*
  default encrypted-arch.conf
  editor 0

``timeout 5`` は、ブートローダーに入ってから、ブートエントリの切り替え
を5秒だけ待ってくれるやつだ。5秒たつとデフォルトのブートエントリに入っ
ていく。デフォルトのブートエントリは::

  # cat /boot/loader/entries/encrypted-arch.conf
  title Arch Linux Encrypted
  linux /vmlinuz-linux
  initrd /intel-ucode.img
  initrd /initramfs-linux.img
  options luks.uuid=1cf54a61-cd17-43e3-ad00-bf94c29dc922 luks.name=1cf54a61-cd17-43e3-ad00-bf94c29dc922=crypt-root root=/dev/mapper/crypt-root rw intel_pstate=no_hwp

`↑はdm-cryptのやつ <https://wiki.archlinux.jp/index.php/Dm-crypt/%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E8%A8%AD%E5%AE%9A>`_ をみながら編集した。

起動したときにファイルシステムのパスフレーズを要求されて、 login ttyが出れば成功。::

  # sync
  # systemctl reboot

しかしこれでは後々マウスが動かないので、あとでこのカーネルをAURでアップデートすることになる。

- https://wiki.archlinux.jp/index.php/Systemd-boot


Reference
---------

- `インストールガイド <https://wiki.archlinux.jp/index.php/%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%82%AC%E3%82%A4%E3%83%89>`_
- `Lenovo ThinkPad X1 Carbon (Gen 5) <https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Carbon_(Gen_5)>`_
- `Category:X1 Carbon (5th Gen) <http://www.thinkwiki.org/wiki/Category:X1_Carbon_(5th_Gen)>`_
- `Efficient Encrypted UEFI-Booting Arch Installation <https://gist.github.com/HardenedArray/31915e3d73a4ae45adc0efa9ba458b07>`_
- `Thinkpad X1 Carbon (Gen 4)にArch Linuxをインストールする <https://qiita.com/miy4/items/796c51813417cc90c77f>`_ <- なぜ systemd hook をつけてないのに systemd-boot でブートローダーが作れるんだ？



Arch Linux Installation to machine #2
-------------------------------------

2018/2/4

To my desktop machine, with NVIDIA GeForce 760, and several hetero HDDs.

OS Install
^^^^^^^^^^

Same os above, using many tools to setup Arch Installation, except for:

- No HDD encryption
- No kernel build for ThinkPad; it's just commodity desktop
- Btrfs for /home: ``# mount /mnt/home /dev/sdb`` where sdb is one of Btrfs disks
- Static IP address configuration
- NVIDIA driver

Btrfs setup
^^^^^^^^^^^


See `Using Btrfs with Multiple Devices <https://btrfs.wiki.kernel.org/index.php/Using_Btrfs_with_Multiple_Devices>`_::

  # mkfs.btrfs -d single /dev/sdb /dev/sdc /dev/sdd /dev/sde
  # mount /dev/sde /mnt

単体の例::

  # mkfs.btrfs -d single /dev/sdf
  btrfs-progs v5.6.1
  See http://btrfs.wiki.kernel.org for more information.

  Detected a SSD, turning off metadata duplication.  Mkfs with -m dup if you want to force metadata duplication.
  Label:              (null)
  UUID:               e092a2dc-b65d-45e4-8dee-46529c1949a0
  Node size:          16384
  Sector size:        4096
  Filesystem size:    931.51GiB
  Block group profiles:
    Data:             single            8.00MiB
    Metadata:         single            8.00MiB
    System:           single            4.00MiB
  SSD detected:       yes
  Incompat features:  extref, skinny-metadata
  Checksum:           crc32c
  Number of devices:  1
  Devices:
     ID        SIZE  PATH
      1   931.51GiB  /dev/sdf

マウントしてサブボリュームを作る::

  # mount /dev/sdf /mnt
  # btrfs sub create /mnt/kuenishi
  # chown kuenishi:kuenishi /mnt/kuenishi

いろいろファイルをコピーする::

  $ cd
  $ mv -v * /mnt/kuenishi/
  $ cp -v -r .* /mnt/kuenishi/


サブボリュームを ``/etc/fstab`` に書く::

  UUID=e092a2dc-b65d-45e4-8dee-46529c1949a0       /home           btrfs           rw,relatime,space_cache,subvolid=5,subvol=/     0 0


これで再起動するとホームディレクトリが btrfs のパーティションになっているはずだ。::

  $ sudo btrfs filesystem show /home
  Label: none  uuid: e092a2dc-b65d-45e4-8dee-46529c1949a0
          Total devices 1 FS bytes used 32.64GiB
          devid    1 size 931.51GiB used 36.01GiB path /dev/sdf

以前作った大きめの btrfs ボリュームが ``/data`` にあるものとする。予め
スナップショット用のサブボリュームを作っておく。::

  # btrfs sub create /home/ss
  # btrfs sub create /data/home-backup



今度はバックアップを実行する。シェルにしてしまった。::

  #!/bin/sh

  set -eux

  TS=`date '+%Y%m%d-%H%M%S'`
  btrfs sub snap /home/kuenishi /home/ss/kuenishi-${TS}
  btrfs property set -ts /home/ss/kuenishi-${TS} ro true
  btrfs send /home/ss/kuenishi-${TS} | btrfs receive /data/home-backup
  btrfs sub delete /home/ss/kuenishi-${TS}


これであとは適当にcronにでもしておけばよい。あとは Docker を使っている
場合、レイヤーを btrfs に保存する方法は簡単で、 ``btrfs sub create
/data/docker`` とやってサブボリュームを作っておく。それから、
``/etc/fstab`` に::


  UUID=5fdf336d-09b3-4b98-8e52-d78b72ceedcf       /data           btrfs           rw,relatime,space_cache,subvolid=5,subvol=/     0 0
  UUID=5fdf336d-09b3-4b98-8e52-d78b72ceedcf       /var/lib/docker         btrfs           rw,relatime,space_cache,subvol=docker   0 0

とかいておくと、 ``/data`` にも使われているボリュームを名前を変えて
``/var/lib/docker`` に置き換えて使ってくれる。Dockerの方でも、btrfs で
あることを検知して勝手にoverlayしてくれるようになる。::

  $ df -h /data /var/lib/docker
  Filesystem      Size  Used Avail Use% Mounted on
  /dev/sdb        1.8T  461G  784G  37% /data
  /dev/sdb        1.8T  461G  784G  37% /var/lib/docker

マシンを再起動する前に一度 ``rm -rf /var/lib/docker/*`` を実行して綺麗
にしておいた方がよいだろう。ちなみにしばらく使ったあとに ``btrfs sub
list /data`` すると大量のレイヤーが表示されて、Dockerが使われているこ
とがよくわかる。

Static IP address
^^^^^^^^^^^^^^^^^^

`Use netctl <https://wiki.archlinux.jp/index.php/Netctl#.E6.9C.89.E7.B7.9A>`_::

  # ip a
  ... (see device name) ...
  # cd /etc/netctl
  # cp examples/ethernet-static enp4s0
  # emacs enp4s0
  ...
  # cat enp4s0
  Interface=enp1s0
  Connection=ethernet
  IP=static
  Address=('10.1.10.2/24')
  Gateway='10.1.10.1'
  DNS=('10.1.10.1')
  # netctl start enp4s0   // to start now
  # netctl enable enp4s0  // to enable it on startup

Don't do any fucking typoes here. Troutbleshoot with ``# journalctl -xe``, like `this <https://bbs.archlinux.org/viewtopic.php?id=161263>`_ .


Nvidia driver
^^^^^^^^^^^^^^

Nouvearu is kind of general and just works - but was too slow for me
at runtime. With `official guide
<https://wiki.archlinux.org/index.php/NVIDIA>`_ you can find your device version::

  $ lspci -k | grep -A 2 -E "(VGA|3D)"

Install yaourt
^^^^^^^^^^^^^^^

Obsolete... yaourt is not maintained any more.



General Trouble Shooting
----------------------------

Locale再生成
^^^^^^^^^^^^^^^^^^^^^^^

ロケールが変になったらいつでも作り直すことができる。
``/etc/locale.gen`` を編集して、必要なロケールのコメントアウトをとりの
ぞく。そのあと再度::

  $ sudo locale-gen

と実行することでロケールが作成される。

- https://wiki.archlinux.jp/index.php/%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB


Mac Mini mid 2011 にインストール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2018/9の時点で `mid 2011 <https://apple-history.com/mac_mini_mid_11>`_
のモデルが最後のアップデート。もうMacOSは動かないくらいのスペックなの
で、 Arch Linux を入れた。 `ブート USB を指して、 Options キーを押しな
がら起動するとブートセレクタに入れる
<https://support.apple.com/ja-jp/HT202796>`_ 。 WiFi はBCM43xxなので、
ブート用のISOには入っていないので、有線ネットワークのアクセスがインストールのためには必要。


インストール後は有線がセットアップされていないので::

  $ sudo pacman -S dhcpcd
  $ sudo systemctl start dhcpcd
  $ sudo systemctl enable dhcpcd


WiFiを使うために

- `yay <https://aur.archlinux.org/packages/yay/>`_ をインストール
- `b43 <https://wiki.archlinux.org/index.php/broadcom_wireless#b43>`_ をインストール
- ``broadcom-wl`` もインストールして再起動。 dmesg で BCM4331 が見えていることを確認
- ``sudo cp /etc/netctl/examples/wireless-wpa /etc/netctl/<profile-name>`` をやって内容を編集
- デバイス名は ``ip link`` で一覧して結果を確認。それらの内容を ``/etc/netctl/<profile-name>`` に書き込む
- ``sudo systemctl start netctl; sudo netctl start <profile-name>``
- `netctl <https://wiki.archlinux.jp/index.php/Netctl>`_
- 多分これでいける


xorg入れたけどなぜかGUIがでてこない
