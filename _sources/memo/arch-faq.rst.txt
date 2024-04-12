#####################################
Arch Linux (FAQ)
#####################################



TODO
----

- いつの間にかF1~F12 のキーのうちいくつかが使えるようになっていた。不思議
- YubiKeyを入れるとキーボードのCaps Lockが戻ってしまう
- `OpenPGP のスマートカード有効化 <https://wiki.archlinux.jp/index.php/Yubikey#OpenPGP_.E3.82.B9.E3.83.9E.E3.83.BC.E3.83.88.E3.82.AB.E3.83.BC.E3.83.89.E3.83.A2.E3.83.BC.E3.83.89.E3.82.92.E6.9C.89.E5.8A.B9.E5.8C.96>`_

Reference
---------

- `インストールガイド <https://wiki.archlinux.jp/index.php/%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%82%AC%E3%82%A4%E3%83%89>`_
- `Lenovo ThinkPad X1 Carbon (Gen 5) <https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Carbon_(Gen_5)>`_
- `Category:X1 Carbon (5th Gen) <http://www.thinkwiki.org/wiki/Category:X1_Carbon_(5th_Gen)>`_
- `Efficient Encrypted UEFI-Booting Arch Installation <https://gist.github.com/HardenedArray/31915e3d73a4ae45adc0efa9ba458b07>`_
- `Thinkpad X1 Carbon (Gen 4)にArch Linuxをインストールする <https://qiita.com/miy4/items/796c51813417cc90c77f>`_ <- なぜ systemd hook をつけてないのに systemd-boot でブートローダーが作れるんだ？





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


急にWiFiがつながらなくなったら
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

パソコンのハードスイッチからオフになっている可能性がある。推し間違いなど。
``rfkill`` で状態を確認できるので、それ経由でオンにするとつながる。
