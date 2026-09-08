# Find the Flag14 Password

When we log in as `level14`, there is no level-specific executable in the home directory.

```sh
level14@SnowCrash:~$ ll
total 12
dr-x------ 1 level14 level14  100 Mar  5  2016 ./
d--x--x--x 1 root    users    340 Aug 30  2015 ../
-r-x------ 1 level14 level14  220 Apr  3  2012 .bash_logout*
-r-x------ 1 level14 level14 3518 Aug 30  2015 .bashrc*
-r-x------ 1 level14 level14  675 Apr  3  2012 .profile*
```

Since there is no local binary to inspect, start with `/bin/getflag`:

```sh
level14@SnowCrash:~$ getflag
Check flag.Here is your token :
Nope there is no token here for you sorry. Try again :)

level14@SnowCrash:~$ gdb /bin/getflag
[...]
(gdb) r
Starting program: /bin/getflag
You should not reverse this
[Inferior 1 (process 26987) exited with code 01]
```

When `/bin/getflag` is run under GDB, it prints `You should not reverse this` and exits. This shows that the program has an anti-debugging check, but this output alone does not prove exactly how `ptrace()` is used internally.

Now inspect the visible strings in the binary:

```sh
[...]
Check flag.Here is your token :
You are root are you that dumb ?
I`fA>_88eEd:=`85h0D8HE>,D
7`4Ci4=^d=J,?>i;6,7d416,7
<>B16\AD<C6,G_<1>^7ci>l4B
B8b:6,3fj7:,;bh>D@>8i:6@D
?4d@:,C>8C60G>8:h:Gb4?l,A
G8H.6,=4k5J0<cd/D@>>B:>:4
H8B8h_20B4J43><8>\ED<;j@3
78H:J4<4<9i_I4k0J^5>B1j`9
bci`mC{)jxkn<"uD~6%g7FK`7
Dc6m~;}f8Cj#xFkel;#&ycfbK
74H9D^3ed7k05445J0E4e;Da4
70hCi,E44Df[A4B/J@3f<=:`D
8_Dw"4#?+3i]q&;p6 gtw88EC
boe]!ai0FB@.:|L6l@A?>qJ}I
g <t61:|4_|!@IF.-62FH&G~DCK/Ekrvvdwz?v|
Nope there is no token here for you sorry. Try again :)
00000000 00:00 0
LD_PRELOAD detected through memory maps exit ..
;*2$"$
GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
.symtab
[...]
```

The unusual strings are not plain hashes. They are obfuscated strings used as inputs to decoding logic before output.

Go back to GDB and inspect the available function symbols:

```sh
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x08048444  _init
0x08048490  strdup
0x08048490  strdup@plt
0x080484a0  __stack_chk_fail
0x080484a0  __stack_chk_fail@plt
0x080484b0  getuid
0x080484b0  getuid@plt
0x080484c0  fwrite
0x080484c0  fwrite@plt
0x080484d0  getenv
0x080484d0  getenv@plt
0x080484e0  puts
0x080484e0  puts@plt
0x080484f0  __gmon_start__
0x080484f0  __gmon_start__@plt
0x08048500  open
0x08048500  open@plt
0x08048510  __libc_start_main
0x08048510  __libc_start_main@plt
0x08048520  fputc
0x08048520  fputc@plt
0x08048530  fputs
0x08048530  fputs@plt
0x08048540  ptrace
0x08048540  ptrace@plt
0x08048550  _start
0x08048580  __do_global_dtors_aux
0x080485e0  frame_dummy
0x08048604  ft_des
0x0804871c  syscall_open
0x0804874c  syscall_gets
0x080487be  afterSubstr
```

The interesting function is `ft_des`. The binary contains several static call sites to `ft_des`, each associated with a different encoded string. That does not mean all of them are called during one execution. Only the branch selected by the UID check is executed.

Set a breakpoint at `main+67`, where the program stops before exiting because of the anti-debugging path, then jump to the block that handles UID `3014`:

```sh
(gdb) b *main+67
Breakpoint 1 at 0x8048989
(gdb) r
Starting program: /bin/getflag

Breakpoint 1, 0x08048989 in main ()
(gdb) ju *main+1183
Continuing at 0x8048de5.
7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 27131) exited normally]
```

Here, `r` is GDB's abbreviation for `run`, and `ju` is an abbreviation for `jump`. The `jump *main+1183` command does not call a function and does not change the real UID. It changes the current instruction pointer to `0x08048de5` and continues execution from there.

The target address is not chosen simply because it is the last `ft_des` call. It is chosen because the UID `3014` branch jumps to `main+1183`, and that block decodes an obfuscated string and prints the result.

The relevant block is:

```asm
   0x08048de5 <+1183>: mov    0x804b060,%eax
   0x08048dea <+1188>: mov    %eax,%ebx
   0x08048dec <+1190>: movl   $0x8049220,(%esp)
   0x08048df3 <+1197>: call   0x8048604 <ft_des>
   0x08048df8 <+1202>: mov    %ebx,0x4(%esp)
   0x08048dfc <+1206>: mov    %eax,(%esp)
   0x08048dff <+1209>: call   0x8048530 <fputs@plt>
[...]
```

The instruction `mov 0x804b060,%eax` reads a value from memory at `0x804b060`. It is different from `mov $0x804b060,%eax`, which would place the address itself into `eax`.

The value read from `0x804b060` is saved in `ebx`, then later passed as the second argument to `fputs`. Since `fputs` takes `(string, stream)`, this is the output stream argument. The exact global variable name is not known from this disassembly alone.

The encoded string argument is prepared here:

```asm
0x08048dec <+1190>: movl   $0x8049220,(%esp)
0x08048df3 <+1197>: call   0x8048604 <ft_des>
```

This places `0x8049220` as the first argument to `ft_des`. The return value from `ft_des` is placed in `eax`, and then passed as the first argument to `fputs`:

```asm
0x08048df8 <+1202>: mov    %ebx,0x4(%esp)
0x08048dfc <+1206>: mov    %eax,(%esp)
0x08048dff <+1209>: call   0x8048530 <fputs@plt>
```

So the confirmed data flow is:

```text
0x8049220 encoded string -> ft_des -> eax decoded string -> fputs
```

Check the actual string stored at `0x8049220`:

```sh
(gdb) x/s 0x8049220
0x8049220:       "g <t61:|4_|!@IF.-62FH&G~DCK/Ekrvvdwz?v|"
```

This matches the last obfuscated string found with `strings`. The `x/s` command confirms the string content at that address; the `movl $0x8049220,(%esp)` instruction and the following `call ft_des` confirm that this string is passed as the first argument to `ft_des`.

Now confirm why UID `3014` is connected to `main+1183`. We do not need the full disassembly here; the relevant comparison and branch are enough:

```sh
(gdb) x/30i 0x08048b85
   [...]
   0x8048bb6 <main+624>:        cmp    $0xbc6,%eax
   0x8048bbb <main+629>:        je     0x8048de5 <main+1183>
   [...]
```

`0xbc6` is `3014` in decimal. This proves that when the UID value in `eax` is `3014`, execution jumps to `main+1183`.

Earlier in `main`, the result of `getuid()` is returned in `eax`, stored at `0x18(%esp)` around `main+444`, and later loaded back into `eax` for these comparisons. Therefore, the relevant execution path is:

```text
getuid() -> UID value in eax -> compare with 3014 -> main+1183
-> 0x8049220 encoded string -> ft_des -> eax decoded string -> fputs
```

The final token output is confirmed by the GDB jump experiment. The disassembly confirms the UID branch and argument flow; the successful execution confirms that starting at `main+1183` prints the decoded token and exits normally.
