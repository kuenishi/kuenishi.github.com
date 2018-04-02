ErgoDox EZ
----------

2017/12/7

- Buy -> `ErgoDox-ez.com <https://ergodox-ez.com/>`_ ~330USD, Import tax 2500JPY
- 3 weeks expected; (11/21 buy -> 12/6 arrive)
- It works out of the box; while...

Build your own firmware::

  (mac)$ sudo port install avr-libc
  (arch)$ yaourt -S avr-libc avr-gcc
  $ git clone git@github.com:kuenishi/qmk_firmware
  $ cd qmk_firmware
  $ make ergodox_ez:default
  $ ls ergodox_ez_default.hex
  ergodox_ez_default.hex

I configured my keymap as `Kinesis-like keymap <https://github.com/kuenishi/qmk_firmware/blob/kinesislike/keyboards/ergodox_ez/keymaps/kinesislike/keymap.c>`_ . To build it::

  $ git clone git@github.com:kuenishi/qmk_firmware
  $ cd qmk_firmware
  $ git checkout kinesislike
  $ make ergodox_ez:kinesislike
  $ ls ergodox_ez_kinesislike.hex
  ergodox_ez_kinesislike.hex

Burn it! See `doc <https://github.com/qmk/qmk_firmware/tree/master/keyboards/ergodox_ez>`_ ::

  (arch)$ yaourt -S teensy-loader-cli
  $ teensy_loader_cli -mmcu=atmega32u4 -w ergodox_ez_kinesislike.hex

And press reset button. For macOS, `downlaod the application <https://www.pjrc.com/teensy/loader.html>`_


- https://ergodox-ez.com/pages/our-firmware.
