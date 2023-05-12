Arch Linux ops (LuKS)
---------------------

Arch Linux de LuKS

Set up a new drive ( say ``/dev/sdc`` ). Create ``/dev/sdc1`` with ``fdisk`` with full volume::

  # fdisk /dev/sdc
  ...
  # fdisk -l /dev/sdc
  ディスク /dev/sdc: 931.51 GiB, 1000204886016 バイト, 1953525168 セクタ
  ディスク型式: SanDisk SSD PLUS
  単位: セクタ (1 * 512 = 512 バイト)
  セクタサイズ (論理 / 物理): 512 バイト / 512 バイト
  I/O サイズ (最小 / 推奨): 512 バイト / 512 バイト
  ディスクラベルのタイプ: gpt
  ディスク識別子: 926D429E-4D84-1842-8FD8-9476CEE3DEB2

  デバイス   開始位置   終了位置     セクタ サイズ タイプ
  /dev/sdc1      2048 1953523711 1953521664 931.5G Linux ファイルシステム
 


Install cryptsetup and btrfs::

  # yay -S cryptset btrfs-progs
  # cryptsetup luksFormat /dev/sdc1
  ...
  Are you sure? (Type uppercase yes): YES
  Enter passphrase:
  Verify passphrase:
  # cryptsetup open /dev/sdc1 data-tank
  Enter passphrase for /dev/sdc1:
  # mkfs.btrfs /dev/mapper/data-tank
  ....

  # mount /mnt/ /dev/mapper/datatank


Check the UUID with lsblk::

  NAME     FSTYPE      FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
  ...
  sdc
  └─sdc1   crypto_LUKS 2           da0d9d95-8ca2-404f-9db5-2717aa429d80
    └─datatank btrfs                   00b1bf82-f7eb-405f-8f8b-c6c7bd389c18  360.1G    61% /mnt


If it's not for boot partition, then let it sit in ``/etc/crypttab`` to make it visible in fstab.::


  # sudoedit /etc/crypttab
  ...
  # tail -n 1
  datatank         UUID=da0d9d95-8ca2-404f-9db5-2717aa429d80    none


Test it out with reboot and make sure ``/dev/mapper/datatank`` exists.
Afterwards, set the UUID of btrfs as ``00b1bf82-f7eb-405f-8f8b-c6c7bd389c18`` like this::

  # sudoedit /etc/fstab
  $ tail -n 1 /etc/fstab
  UUID=00b1bf82-f7eb-405f-8f8b-c6c7bd389c18       /datatank   btrfs   rw,relatime     0 0
