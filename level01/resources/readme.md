# Find the Flag01 Password

In the home directory, there are no files or binaries to exploit, nor are there any files owned by `flag01`.

By looking at `/etc/passwd`, we can check the user account's basic information. It also reveals the encrypted password.

It looks like this:

```sh
cat /etc/passwd
```

```text
[...]
flag01:42....:3001:3001::/home/flag/flag01:/bin/bash
[...]
```

Because `/etc/passwd` is readable, it can be copied.
Copy it from the VM to the local machine for further transfer to Kali or another OS.

Use JTR (John the Ripper) to crack the password without copying the file.
