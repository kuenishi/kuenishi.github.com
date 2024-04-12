#####################################
Arch Linux Setup (GUI)
#####################################

準備
------

(今はやらなくていいかも)このままだと `トラックポイント/トラックパッドが機能しない
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

GUI
----

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
く上位でやりたいので今ここ。また、ログイン時に設定されるようにしておくため、 ``xfce4-settings-manager`` から *Session and Startup* > *Application Autostart* タブにいって、 *Add* から追加する。[ `Link <https://serverfault.com/a/1005911>`_ ]

DPIはAppearance （外観）から変更。とりあえず字大きめの160で。

Weird display vibrarion
^^^^^^^^^^^^^^^^^^^^^^^

Sometimes a screen vibrates suddenly with a dmesg line including ``[drm:intel_cpu_fifo_underrun_irq_handler [i915]] *ERROR* CPU pipe A FIFO underrun``  ... which may be solved with installing ``xf86-video-intel``  .

- `Linux console goes crazy ("cpu pipe a fifo underrun") when Xorg quits. <https://bbs.archlinux.org/viewtopic.php?id=198157>`_


日本語入力
---------------

`Japanese fonts <https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88#.E6.97.A5.E6.9C.AC.E8.AA.9E>`_ ::

  $ sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
  $ sudo pacman -S fcitx5-im fcitx5-mozc fcitx5-configtool
  $ echo 'export GTK_IM_MODULE=fcitx5
  export XMODIFIERS=@im=fcitx5' > ~/.xprofile
  $ echo 'export GTK_IM_MODULE=fcitx5
  export XMODIFIERS=@im=fcitx5' > ~/.bash_profile


で一旦ログアウトして、いろいろGUIをいじっていたら動いた。もしかしたら::

  $ fcitx5-autostart

が必要かも。どうにも変だったら ``fcitx5-diagnose`` こまんどで原因調査するとよい。 `ろけーるが変だった <https://wiki.archlinux.jp/index.php/ロケール>`_ ことがある。See also: `Arch Linuxの日本語入力をfcitxからfcitx5に切り替える <https://note.kurodigi.com/archlinux-fcitx5/>`_

Properly render PDF without embedded Japanese fonts::

  $ sudo pacman -S poppler-data

System Update +AUR
-------------------

システムアップデートは ::

  $ sudo pacman -Syu

このときAURで入れていたパッケージに注意すること。


パッケージを探すのは::

  $ pacman -Ss <pkgname>

yay を使うとAURもいける::

  $ cd /tmp
  $ git clone https://aur.archlinux.org/yay.git
  $ cd yay
  $ makepkg -si
  $ yay



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

  $ sudo pacman -S alsa-utils pipewire-pulse
  $ alsamixer

これだけで音が鳴るようになる。 `XfceのUI
<https://wiki.archlinux.org/index.php/xfce#Keyboard_volume_buttons>`_
や、ThinkPadのF1~F3 でのコントロールはできていない。Xfce4のPulseAudio
pluginは ``pavucontrol`` をインストールして再起動したら使えるように
なった。 PulseAudio から Pipewire への移行が進んでいるらしい。

-> https://wiki.archlinux.org/index.php/List_of_applications#Volume_managers


WiFi
----

NetworkManagerを入れるだけで十分。::

  $ sudo pacman -S networkmanager
  $ sudo systemctl enable NetworkManager
  $ sudo systemctl start NetworkManager


WiFiもLANも各種設定は ``nmtui(1)`` で実施する。操作は ``nmcli(1)`` でもできる。


マルチ（外部）ディスプレイ
-------------------------------

家のASUSの4KディスプレイはHDMIを刺すだけで特になにもなく 3000x2000 く
らいで認識されて映った。なにかあったら `マルチディスプレイ
<https://wiki.archlinux.jp/index.php/%E3%83%9E%E3%83%AB%E3%83%81%E3%83%87%E3%82%A3%E3%82%B9%E3%83%97%E3%83%AC%E3%82%A4>`_
を参照。
