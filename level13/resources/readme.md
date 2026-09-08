# Find the Flag13 Password

When we log in as `level13`, we can see the `level13` executable in the home directory.

Run it first:

```sh
level13@SnowCrash:~$ ./level13
UID 2013 started us but we we expect 4242
level13@SnowCrash:~$ ltrace ./level13
__libc_start_main(0x804858c, 1, 0xbffff7f4, 0x80485f0, 0x8048660 <unfinished ...>
getuid()                                                     = 2013
getuid()                                                     = 2013
printf("UID %d started us but we we expe"..., 2013UID 2013 started us but we we expect 4242
)          = 42
exit(1 <unfinished ...>
+++ exited (status 1) +++
```

The program checks the current UID with `getuid()`. Our UID is `2013`, but the program expects `4242`, so it exits before printing the token.

Inspect the binary with `gdb`:

```sh
gdb ./level13

(gdb) disassemble main
Dump of assembler code for function main:
   0x0804858c <+0>:     push   %ebp
   0x0804858d <+1>:     mov    %esp,%ebp
   0x0804858f <+3>:     and    $0xfffffff0,%esp
   0x08048592 <+6>:     sub    $0x10,%esp
   0x08048595 <+9>:     call   0x8048380 <getuid@plt>
   0x0804859a <+14>:    cmp    $0x1092,%eax
   0x0804859f <+19>:    je     0x80485cb <main+63>
   0x080485a1 <+21>:    call   0x8048380 <getuid@plt>
   0x080485a6 <+26>:    mov    $0x80486c8,%edx
   0x080485ab <+31>:    movl   $0x1092,0x8(%esp)
   0x080485b3 <+39>:    mov    %eax,0x4(%esp)
   0x080485b7 <+43>:    mov    %edx,(%esp)
   0x080485ba <+46>:    call   0x8048360 <printf@plt>
   0x080485bf <+51>:    movl   $0x1,(%esp)
   0x080485c6 <+58>:    call   0x80483a0 <exit@plt>
   0x080485cb <+63>:    movl   $0x80486ef,(%esp)
   0x080485d2 <+70>:    call   0x8048474 <ft_des>
   0x080485d7 <+75>:    mov    $0x8048709,%edx
   0x080485dc <+80>:    mov    %eax,0x4(%esp)
   0x080485e0 <+84>:    mov    %edx,(%esp)
   0x080485e3 <+87>:    call   0x8048360 <printf@plt>
   0x080485e8 <+92>:    leave
   0x080485e9 <+93>:    ret
End of assembler dump.
```

The important part is:

```asm
0x08048595 <+9>:     call   0x8048380 <getuid@plt>
0x0804859a <+14>:    cmp    $0x1092,%eax
0x0804859f <+19>:    je     0x80485cb <main+63>
```

After `getuid()` returns, the UID is stored in `$eax`. The program compares `$eax` with `0x1092`, which is `4242` in decimal. If the values match, execution jumps to the code that calls `ft_des()` and prints the token.

We can use `gdb` to stop after the check fails and replace `$eax` with the expected value:

```sh
(gdb) break *0x080485a1
Breakpoint 1 at 0x80485a1
(gdb) run
Starting program: /home/user/level13/level13

Breakpoint 1, 0x080485a1 in main ()

(gdb) display/d $eax
1: /d $eax = 2013
(gdb) set $eax=0x1092
(gdb) display/d $eax
2: /d $eax = 4242
(gdb) continue
Continuing.
your token is 2A31L79asukciNyi8uppkEuSx
[Inferior 1 (process 26399) exited with code 050]
```

The breakpoint is set at `0x080485a1`, just after the failed comparison path begins. At that point, `$eax` still contains our real UID, `2013`. Setting `$eax` to `0x1092` changes it to `4242`, so the program continues as if the UID check had passed.

After continuing execution, the program prints the token for the next level.
