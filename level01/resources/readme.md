# Find the Flag01 Password

In the home directory, there are no files or binaries to exploit, nor are there any files owned by `flag01`.

By looking at `/etc/passwd`, we can check the user account's basic information. In this case, it also reveals the password hash for `flag01`.

It looks like this:

```sh
cat /etc/passwd
```

```text
[...]
level12:x:2012:2012::/home/user/level12:/bin/bash
level13:x:2013:2013::/home/user/level13:/bin/bash
level14:x:2014:2014::/home/user/level14:/bin/bash
flag00:x:3000:3000::/home/flag/flag00:/bin/bash
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
flag02:x:3002:3002::/home/flag/flag02:/bin/bash
flag03:x:3003:3003::/home/flag/flag03:/bin/bash
[...]
```

Because `/etc/passwd` is readable, it can be copied. Or you can remember the hash.

Normally, the second field contains `x`, indicating that the password hash is stored elsewhere, typically in `/etc/shadow`. Let's give it a look.
```bash
level01@SnowCrash:~$ cat /etc/shadow
cat: /etc/shadow: Permission denied
``` 
Unfortunately, we can't see the password.

But for `flag01`, the hash is stored directly in this field. Its 13-character format is consistent with traditional Unix DES crypt, which uses a two-character salt followed by an 11-character hash.

Use John the Ripper to inspect the recovered password without copying the full file.

Unfortunately, we can not install JTR on the VM. So we are going to use our own machine.

```bash
echo 42hDRfypTqqnw > password
john --show password
?:abcdefg
```

Here, `--show` displays a password that John already has available for the given hash. It is not the same as running a new cracking attempt.

This is the password for `flag01`. Don't forget to log in as `flag01` and run `getflag`.
