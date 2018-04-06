########
ZFS Memo
########

2017/11/3 - 11/5 FreeBSD Update from 10.3 -> 11.1 のためにバックアップ。

プール ``data`` があって、パーティション ``data`` と ``data/user`` が
いまのところある。 ``data`` にはTimeMachineデータが入っているだけなの
で特に退避とかはしない。 ``data/user`` だけバックアップする。スナップ
ショットの作成は::
  
  # zfs snapshot data/user@10to11
  
スナップショットを別ドライブ上にあるプール ``attic`` に流し込む::
  
  # zfs send data/user@10to11 | zfs recv attic                           

このとき ``attic`` 上（特に ``/attic/user`` というディレクトリ）にデー
タを置かないこと。置くなら、 ``-F`` オプションで上書きする必要がある。
スナップショットの転送が終わったら::

  # zfs list -t snapshot

で存在とサイズの一致を確認。ファイルで保存したい場合は::

  # zfs send data/user@10to11 | gzip > data-user-10to11.gz               

などとする。転送元のスナップショットを削除するには::

  # zfs destroy data/user@10to11
  
ちなみに、``data`` というプールがあって ``data/user`` を
作りたい場合は::

  # zfs create data/user

肝心のアップデートは::

  # freebsd-update fetch              
  # freebsd-update install            
  # : > /bin/sbin/bspatch
  # freebsd-update upgrade -r 11.0-RELEASE                               
  # freebsd-update install
  <reboot>
  # freebsd-update install
  # pkg update
  # pkg upgrade
  # freebsd-update install

`See <https://www.freebsd.org/releases/11.0R/announce.html>`_ for more
details, which has upgrade precedure a bit.
