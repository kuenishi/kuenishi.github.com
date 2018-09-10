#####################################
Arch Linux Installation & Maintenance
#####################################

2017/10/19 - 2017/10/20

2018/9/9 Mac mini mid 2011 のインストールを追加
2018/3/15 カーネルパッチの部分を削除
2018/4/6 Trouble Shooting追加

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
で、消しておくこと。で、最後にベースシステムを ``pacstrap`` でインストー
ルする::

  # grep Japan -A 1 /etc/pacman.d/mirrorlist > mirrorlist
  (remove  -- here)
  # vi mirrorlist
  # mv mirrorlist /etc/pacman.d/
  # pacstrap /mnt base base-devel

この時点で ``/etc/fstab`` を作ったり、基本的なファイルを先においておく。::

  # genfstab -U /mnt >> /mnt/etc/fstab
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

GUI
----

まずは基本的なユーザー作成::

  # mkdir /home/kuenishi
  # chown kuenishi:kuenishi /home/kuenishi
  # useradd kuenishi
  # passwd kuenishi
  # vigr -> add kuenishi to wheel
  # pacman -S sudo
  # visudo -> enable wheel as ALL=ALL
  $ sudo pacman -S git


このままだと `トラックポイント/トラックパッドが機能しない
<https://wiki.archlinux.jp/index.php/Lenovo_ThinkPad_X1_Carbon_(Gen_5)#.E3.83.88.E3.83.A9.E3.83.83.E3.82.AF.E3.83.9D.E3.82.A4.E3.83.B3.E3.83.88.2F.E3.83.88.E3.83.A9.E3.83.83.E3.82.AF.E3.83.91.E3.83.83.E3.83.89.E3.81.8C.E6.A9.9F.E8.83.BD.E3.81.97.E3.81.AA.E3.81.84>`_
という問題にぶつかるので、それを回避するため設定を入れる。
``psmouse.synaptics_intertouch=1`` をカーネルパラメータに追加する。カー
ネルパラメータを設定するには
``/boot/loader/entries/encrypted-arch.conf`` を書き換える。カーネルと
initrd の新しいファイルが ``/boot`` に置かれているので、両方を置き換え
たら再起動。たとえばこんな感じ(``-tp-x1-carbon-5th`` がついているもの
は 4.14 からはもう不要になった)::

  # ls /boot
  EFI                                            initramfs-linux-tp-x1-carbon-5th.img  loader
  initramfs-linux-fallback.img                   initramfs-linux.img                   vmlinuz-linux
  initramfs-linux-tp-x1-carbon-5th-fallback.img  intel-ucode.img                       vmlinuz-linux-tp-x1-carbon-5th
  # cat /boot/loader/entries/encrypted-arch.conf
  title Arch Linux Encrypted
  linux /vmlinuz-linux
  initrd /intel-ucode.img
  initrd /initramfs-linux.img
  options luks.uuid=1cf54a61-cd17-43e3-ad00-bf94c29dc922 luks.name=1cf54a61-cd17-43e3-ad00-bf94c29dc922=crypt-root root=/dev/mapper/crypt-root rw intel_pstate=no_hwp psmouse.synaptics_intertouch=1

Enlightenment を少し試してみたが、使いにくいので xfce4 に戻る。インストールと起動テスト::

  $ sudo pacman -S xorg-server xfce4 xfce4-goodies
  $ startxfce4

これで起動を確認する。変なことになって失敗したら ``Ctrl+Alt+Del`` で戻
れる。lightdm は最初失敗。ログをみるのもだるいので sddm に。  `SDDM を
有効にするには
<https://wiki.archlinux.jp/index.php/%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4%E3%83%9E%E3%83%8D%E3%83%BC%E3%82%B8%E3%83%A3>`_ ::

  $ sudo pacman -S sddm
  $ sudo systemctl enable sddm

これでリブートしてみる。テストしたければ enable 前に ``sudo systemctl start sddm`` で。
CtrlとCaps Lockを入れ替えるのは、とりあえず::

  $ echo 'setxkbmap -option ctrl:nocaps' > swapctrlcaps.sh
  $ chmod 755 swapctrlcaps.sh
  $ ./swapctrlcaps.sh

で Caps lock を Ctrl にする。他にもキーボードマッピングをいじる方法と
かがあるけど、複数のものが絡まって複雑になると困ってしまうので、なるべ
く上位でやりたいので今ここ。

DPIはAppearance （外観）から変更。とりあえず字大きめの160で。

Weird display vibrarion
^^^^^^^^^^^^^^^^^^^^^^^

Sometimes a screen vibrates suddenly with a dmesg line including ``[drm:intel_cpu_fifo_underrun_irq_handler [i915]] *ERROR* CPU pipe A FIFO underrun``  ... which may be solved with installing ``xf86-video-intel``  .

- `Linux console goes crazy ("cpu pipe a fifo underrun") when Xorg quits. <https://bbs.archlinux.org/viewtopic.php?id=198157>`_


日本語入力
---------------

`Japanese fonts <https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88#.E6.97.A5.E6.9C.AC.E8.AA.9E>`_ ::

  $ sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra adobe-source-han-sans-jp-fonts otf-ipafont ttf-sazanami ttf-hanazono
  $ sudo pacman -S fcitx-im fcitx-mozc fcitx-configtool
  $ echo 'export GTK_IM_MODULE=fcitx
  export XMODIFIERS=@im=fcitx' > ~/.xprofile


で一旦ログアウトして、いろいろGUIをいじっていたら動いた。もしかしたら::

  $ fcitx-autostart

が必要かも。どうにも変だったら ``fcitx-diagnose`` こまんどで原因調査するとよい。 `ろけーるが変だった <https://wiki.archlinux.jp/index.php/ロケール>`_ ことがある。

Properly render PDF without embedded Japanese fonts::

  $ sudo pacman -S poppler-data

System update
-------------

システムアップデートは ::

  $ sudo pacman -Syu

このときAURで入れていたパッケージに注意すること。


パッケージを探すのは::

  $ pacman -Ss <pkgname>


Installed Application
---------------------

いろんなアプリを入れる::

  $ sudo pacman -S tmux firefox emacs slock
  $ sudo pacman -S bash-completion

Spotlightのようなアプリケーションランチャーがあると便利だなと思ってい
たが `Whisker Menu
<http://www.webupd8.org/2014/06/xfce-app-launcher-whisker-menu-sees-new.html>`_
を `Windowsキーに割り当て
<http://thinkonbytes.blogspot.jp/2014/04/xubuntu-open-whisker-menu-with-windows.html>`_
て一見落着。

`Slock <https://wiki.archlinux.jp/index.php/Slock>`_ is just a screen
locker. Type 'slock' when you leave your seat temporariliy or for a
long time. -> `Other screen lockers <https://wiki.archlinux.org/index.php/List_of_applications/Security#Screen_lockers>`_

``bash-completion`` があれば、ターミナルに出た適当な長さのワードを拾って補完してくれるので特にzshとかいらない。

AURでは ``yaourt`` だけを入れる。 ``package-query`` に依存しているので、それを先に入れる。::

  $ git clone https://aur.archlinux.org/package-query.git
  $ cd package-query
  $ makepkg -is
  $ cd ..
  $ git clone https://aur.archlinux.org/yaourt.git
  $ cd yaourt
  $ makepkg -is
  $ cd ..

いろいろ入れる::

  $ yaourt -S --noconfirm google-chrome slack-desktop gyazo

Sound
-----

`サウンド <https://wiki.archlinux.jp/index.php/Advanced_Linux_Sound_Architecture>`_ は::

  $ sudo pacman -S alsa-utils
  $ alsamixer

これだけで音が鳴るようになる。 `XfceのUI
<https://wiki.archlinux.org/index.php/xfce#Keyboard_volume_buttons>`_
や、ThinkPadのF1~F3 でのコントロールはできていない。Xfce4のPulseAudio
pluginは ``pavucontrol`` をインストールして再起動？したら使えるように
なった。

-> https://wiki.archlinux.org/index.php/List_of_applications#Volume_managers


WiFi
----

`Wicd <https://wiki.archlinux.org/index.php/Wicd>`_ を入れたらそれなりに快適に動く::

  $ sudo pacman -S wicd wicd-gtk gksu python2-notify
  $ sudo systemctl enable wicd
  $ sudo systemctl start wicd

`WiFi <https://wiki.archlinux.jp/index.php/WPA_supplicant>`_ なぜか一
度適当に設定したらいつの間にか動くようになってしまった…　電波強度をパ
ネルに出すのに wavelan というのをつけてある。ツールは他にも
``wifi-menu`` とかいろいろある。->
https://wiki.archlinux.jp/index.php/%E3%83%AF%E3%82%A4%E3%83%A4%E3%83%AC%E3%82%B9%E8%A8%AD%E5%AE%9A

WPA2 Enterpriseでクライアント認証でWiFi APに接続するときに、 EAP-TLSで
接続してPKCS#12の秘密鍵とパスフレーズだけで接続する場合にはなぜかWicd
ではうまく接続できず、結局 ``netctl`` でやった。::

  $ sudo systemctl stop wicd
  $ sudo systemctl disable wicd
  $ sudo pacman -R wicd wicd-gtk
  $ sudo pacman -S netctl
  $ sudo netctl list
  $ sudo wifi-menu wlp4s0
  (...find the SSID of WPA2 Enterprise and try to setup...)
  $ sudoedit /etc/netctl/wlp4s0-<ssid>
  $ cat /etc/netctl/wlp4s0-<ssid>
  Description='Automatically generated profile by wifi-menu'
  Interface=wlp4s0
  Connection=wireless
  Security=wpa-configsection
  IP=dhcp
  WPAConfigSection=(
        'ssid="hoge"'
        'key_mgmt=WPA-EAP'
        'eap=TLS'
        'proto=WPA RSN'
        'identity="kuenishi"'
        'private_key="/home/kuenishi/.pki/kuenishi_aho.p12"'
        'private_key_passwd="**********"'
        'priority=1'
        'phase2="auth=PAP"'
  )

このファイルはたしか
``/etc/netctl/examples/wireless-wpa-configsection`` からのコピペだった
気がする。これで接続できたら systemd 設定をする。::

  $ sudo netctl list
  * wlp4s0-hoge
  $ sudo netctl enable wlp4s0-hoge


Docker
------

インストールしたら自分をDockerグループに入れて一旦ログアウト（グループ追加を有効にするため）::

  $ sudo pacman -S docker
  $ vigr
  $ exit (WMもログアウト）

その後Daemonを上げてアクセス::

  $ sudo systemctl start docker
  $ docker run -it ubuntu /bin/bash

GPG keys
--------

GPG key setup esp. pinentry::

  $ cat .gnupg/gpg-agent.conf
  # PIN entry program
  # pinentry-program /usr/bin/pinentry-curses
  # pinentry-program /usr/bin/pinentry-qt
  # pinentry-program /usr/bin/pinentry-kwallet

  pinentry-program /usr/bin/pinentry-gtk-2


`GPG signature は作成またはインポート
<http://kuenishi.hatenadiary.jp/entry/2016/07/12/013041>`_ する。Gitも
それで。::

  $ git config --global gpg.program gpg2
  $ gpg2 --list-secret-keys
  $ git config --global user.signingkey xxxxxxxx
  $ git log --show-signature
  $ git tag -s tagname
  $ git show tagname

- https://www.gnupg.org/gph/en/manual/x334.html
- `baccounts <https://github.com/kuenishi/baccounts>`_ -> GnuPG 2.0ま
  でのフォーマットしか対応してないが、新しい GnuPG だと旧フォーマット
  で出力できる。自前で GnuPG 1.x を入れてると pacman-keyring とかと干
  渉してシステムアップデートできなくなるトラブルがあったので、自前では
  入れずに公式でやること。

ClamAV
------

セキュリティチェックでウイルスソフトが必要なのでインストール::

  $ sudo pacman -S clamav
  $ sudo freshclam
  $ sudo systemctl enable freshclamd.service
  $ sudo systemctl start freshclamd.service
  $ sudo systemctl enable clamd.service
  $ sudo systemctl start clamd.service

実際にウイルス検知ができるかチェックする::

  $ curl http://www.eicar.org/download/eicar.com.txt | clamscan -

以下が出力されれば検出成功です::

  stdin: Eicar-Test-Signature FOUND

`Further info <https://wiki.archlinux.jp/index.php/ClamAV>`_

マルチ（外部）ディスプレイ
-------------------------------

家のASUSの4KディスプレイはHDMIを刺すだけで特になにもなく 3000x2000 く
らいで認識されて映った。なにかあったら `マルチディスプレイ
<https://wiki.archlinux.jp/index.php/%E3%83%9E%E3%83%AB%E3%83%81%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4>`_
を参照。

Windows on VirtualBox
---------------------

`Windows 10 Downlaod <https://www.microsoft.com/ja-jp/software-download/windows10ISO>`_ からダウンロードしつつ、VirtualBoxをインストールする::

  $ sudo pacman -S virtualbox virtualbox-guest-iso vde2

Windows予定のイメージを作ってDVDブートすると ``VT-x is disabled in the
BIOS for all CPU modes`` と叱られたので、BIOSで設定をオンにする。最近
は VT-d とかいうのまであるらしい。

予めWindowsのプロダクトキーは Windows Product Key Viewer というので抜
いておいた。ふつうにインストールできているっぽい。

image:: windows10-product-key-thinkpad-x1c-5th.jpg


YubiKey authentication
----------------------

`Local Authentication Using Challenge Response
<https://developers.yubico.com/yubico-pam/Authentication_Using_Challenge-Response.html>`_
would be enough to prove personality, with 2FA. YubiCloud requires
network connectivity and its not given for free...::

  $ sudo pacman -S yubico-pam
  (insert yubikey)
  $ mkdir ~/.yubico
  $ ykpamcfg -2 -v
  (a file will be saved to ~/.yubico/challenge-xxxxxx)

Move challenge-xxx file to ``/var/yubico`` following instructions
described in the page above; and in Arch ``/etc/pam.d/system-auth`` is
used rather than ``common-auth`` in the document. Before hand, make
sure you have another terminal window with root privilege just in case
to roll back ``system-auth`` file. The edit will be::

  $ sudoedit /etc/pam.d/system-auth
  (add
  'auth   required        pam_yubico.so mode=challenge-response chalresp_path=/var/yubico'
  after a line of pam_unix.so)
  $ cat /etc/pam.d/system-auth
  #%PAM-1.0

  auth      required  pam_unix.so     try_first_pass nullok
  auth      required  pam_yubico.so mode=challenge-response chalresp_path=/var/yubico
  auth      optional  pam_permit.so
  auth      required  pam_env.so
  (.. snip ..)

Wait for moment, and test with ``sudo -s`` which will block after you
input sudo password and the YubiKey is waiting for your touch.

See also:  `ykpamcfg(1) <https://developers.yubico.com/yubico-pam/Manuals/ykpamcfg.1.html>`_


TODO
----

- いつの間にかF1~F12 のキーのうちいくつかが使えるようになっていた。不思議
- YubiKeyを入れるとキーボードのCaps Lockが戻ってしまう
- pacman/AURのフロントエンドを何か選ぶ -> https://wiki.archlinux.org/index.php/AUR_helpers -> とりあえず ``yaourt`` で困っていない。
- `OpenPGP のスマートカード有効化 <https://wiki.archlinux.jp/index.php/Yubikey#OpenPGP_.E3.82.B9.E3.83.9E.E3.83.BC.E3.83.88.E3.82.AB.E3.83.BC.E3.83.89.E3.83.A2.E3.83.BC.E3.83.89.E3.82.92.E6.9C.89.E5.8A.B9.E5.8C.96>`_

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

https://wiki.archlinux.jp/index.php/Yaourt



General Trouble Shooting
------------------------

Locale再生成
~~~~~~~~~~~~~~~

ロケールが変になったらいつでも作り直すことができる。
``/etc/locale.gen`` を編集して、必要なロケールのコメントアウトをとりの
ぞく。そのあと再度::

  $ sudo locale-gen

と実行することでロケールが作成される。

- https://wiki.archlinux.jp/index.php/%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB


Mac Mini mid 2011 にインストール
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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


Pacman のアップデートで GnuPG エラーが出る
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

パッケージアップデート時に ``gnupg: signature from "Gaetan Bisson
<bisson@archlinux.org>" is invalid`` 的なエラーがでて失敗したら、以下
の手順でキーを更新する::

  $ sudo rm /var/lib/pacman/sync/*
  $ sudo pacman-key --init
  $ sudo pacman-key --populate archlinux
  $ sudo pacman-key --refresh-keys


Fix kernel or other files 2018/4/6
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``yaourt -Syu`` が妙に時間がかかったのでマシンを途中で止めてしまったと
ころ、起動時（LUKSのパスワード入力）時にキーボードが効かなくなり、パス
ワードを入力できなくなった。このためその後のブートシーケンスを継続でき
ない。

まずは起動して ``yaourt -Syu`` の続きを完了することを目指す。ブートディ
スクを使って、まずブートディスクのカーネルで起動しておく。それでchroot
環境をつくる。::

  root@archiso ~ # cryptsetup open /dev/nvme0n1p2 crypt-root
  Enter passphrase for /dev/nvme0n1p2:
  root@archiso ~ # mount /dev/mapper/crypt-root /mnt
  root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot
  root@archiso ~ # timedatectl set-ntp true
  root@archiso ~ # timedatectl status
  ...
  # arch-chroot /mnt
  [root@archiso /]# yaourt -Syu

とやると、 ``pacman -Syu`` やってるんじゃないの？といわれる。定番のロッ
クファイルが残っているやつなのでロックファイルを消す。::

  [root@archiso /]# rm /var/lib/pacman/db.lck
  [root@archiso /]# yaourt -Syu
  ...

今度は普通に最後まで動く。というわけでchroot環境を出てから再起動
``shutdown -r now`` してみてもやっぱりパスワードを入力できない（キーボー
ドが使えていない）。はっそういえば…と思いなおして、また同じ手順を繰り
返して chroot 環境に入って、::

  [root@archiso /]# mkinitcpio -p linux
  ...

として、 initrd を作り直して再起動したところ無事にキーボードが使えるよ
うになった。おそらくは vmlinuz (カーネルのファイル) を作っただけで
initrd を再作成していなかったため何かがズレて起動だけはするけどキーボー
ドが使えなくなっていたのだろうと思われる。教訓: カーネルのアップデート
途中に電源長押しで再起動なんかしてはいけないし、 *金曜夕方の退勤間際に
間違ってもOSのアップデートなんかしてはいけない* 。
