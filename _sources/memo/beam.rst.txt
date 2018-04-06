####
BEAM
####

- from https://gist.github.com/kuenishi/3183043


assemble
--------

```
$ erlc -S test.erl
```

registers
---------

- 1024 ``X`` registers
- ``Y`` register

Registers X, Y are represented like this:

:5 bit tag: 27 bit value

The tags are 3=X, 4=Y, 5=r, 6=Literal

- ``CP``

 - "Continuation Pointer" - return address

- ``PC``

 - "Program Counter" - next instruction to be executed

important instructions
----------------------

- test
- move ``{move, Src, Dest}``

64 is the opcode for "move", 0x18000000 is the encoding of argument that means register ``{x, 0}``

- gc_bif
- call ``{call, Arity, {f, N}}``

 - set ``P=N, CP=Next``
 - jumps to label ``N``
 - arguments are stored in X0 ~ Xn

- return

 - does ``PC=CP``
 - return value is stored in X0

- receive

- allocate
- deallocate ``{deallocate, N}``

 - create and remove stack frames.
 - there also exists ``{allocate_zero, Ny, Ng}`` - ``Ny`` Y registers

other instructions
------------------

- label ``{label, N}``

 - Nth label in the module

- function ``{function, Name, Arity, LabelID}``

 - defines a function

- bif ``{bif, FuncAtom, {f, 0}, [], {x, 0}}``

 - calls, ?

- gc_bif ``{gc_bif, '*', {f, A}, N1, [R1, R2, R3]}``

 - jumps to a function named ``'*'``
 - multiply R1 and R2, and set the result into R3. If any failure occurs jump to ``A``. If GC is triggered include N1 X registers starting at X0 in the root set for GC.

- send ``send.``

 - sends the data in ``X0`` ?

- loop_rec ``{loop_rec, {f, N}, {x, 0}}``

 - receive clause starts here, till ``loop_rec_end``

- wait_timeout ``{wait_timeout, {f, N}, {integer, T}}``

- test ``{test, is_eq_exact, {f, N}, [{x,0}, {atom, foo}]}``
- ``remove_message.``

misc:
-----

Context Switch
--------------

No other timing than when a ``call`` or ``receive`` instruction is executed, check a reduction counter.

How to disasemble
-----------------

```erl
beam_lib:chunks("hw.beam", [abstract_code]),
```

Reference
---------

- instruction set? [genop.tab](https://github.com/erlang/otp/blob/master/lib/compiler/src/genop.tab>)
- source code - [beam_emu.c](https://github.com/erlang/otp/blob/master/erts/emulator/beam/beam_emu.c#L1103>)
- [erlang vm opcodes: beam_opcodes.erl](https://github.com/mfoemmel/erlang-otp/blob/master/lib/compiler/src/beam_opcodes.erl>)
- Joe Armsgrong, [Deconstructing the BEAM](http://erlang.org/pipermail/erlang-questions/2012-May/066524.html>) , 2012.
- [The Erlang BEAM Virtual Machine Specification](http://www.cs-lab.org/historical_beam_instruction_set.html>)
- [File format for Beam R5 and later](http://www.erlang.se/~bjorn/beam_file_format.html>)
- [The Evolution of the Erlang VM](http://erlang.org/pipermail/erlang-questions/2012-May/066524.html>)

- [A Guide to the Erlang Source](http://www.trapexit.org/A_Guide_To_The_Erlang_Source>)
- [Erlang source code guide](http://stackoverflow.com/questions/3755506/erlang-source-code-guide>)

- [The Erlang BEAM Virtual Machine Specification](http://www.cs-lab.org/historical_beam_instruction_set.html) 1997.
- [BEAM File Format (pdf)](http://synrc.com/publications/cat/Functional%20Languages/Erlang/BEAM.pdf)  2012.
- [Erlang on Xen: BEAM instruction set](http://erlangonxen.org/more/beam)
