#####################################
Arch Linux (misc)
#####################################


Docker
------

インストールしたら自分をDockerグループに入れて一旦ログアウト（グループ追加を有効にするため）::

  $ sudo pacman -S docker
  $ vigr
  $ exit (WMもログアウト）

その後Daemonを上げてアクセス::

  $ sudo systemctl start docker
  $ docker run -it ubuntu /bin/bash


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

