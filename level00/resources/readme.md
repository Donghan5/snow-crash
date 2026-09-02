# Find the Password for Flag00

## First Check

First of all, we are going to check the home directory.

The result is... empty!

## Second Check

We move on to the next step! We checked `/etc/passwd` but found nothing meaningful. We thought about looking for files owned by `flag00`.

Here is the command to check for them:

```sh
find / -user flag00 2>/dev/null
```

`2>/dev/null` allows the command to run without displaying permission errors. `2` is standard error (`stderr`), and `>` is the redirection operator. This means that all standard error output is redirected to `/dev/null`. When data goes to `/dev/null`, it simply disappears, like something falling into a black hole.

## Result

Two files are owned by the user `flag00`:

```text
/usr/sbin/john
/rofs/usr/sbin/john
```

They both contain the same string:

```sh
cat /usr/sbin/john
cat /rofs/usr/sbin/john
```

```text
cdiiddwpgswtgt
```

Sadly, it is not the password. It seems to be encrypted or encoded.

Using dCode's Cipher Identifier, the string can be identified as a Caesar cipher. After testing the possible shifts, it produces readable plaintext.

But before moving on to the cipher, there is something interesting here:

Why do two apparently identical files exist?

To understand this, we have to know what `/rofs` is.

`/rofs` stands for a read-only filesystem mount in the environment.

A read-only filesystem contains the original system files, while the running filesystem exposes another view of those files through the root directory, `/`.

Conceptually, we can think of it like this:

```text
Read-only base filesystem
/rofs
└── usr
    └── sbin
        └── john
             │
             │ included in the root filesystem view
             ▼
/
└── usr
    └── sbin
        └── john
```

Therefore, `/rofs/usr/sbin/john` represents the file stored in the read-only base filesystem, while `/usr/sbin/john` is the same file as seen through the running system's root filesystem.

This is why searching from `/` can return both paths.

### Read-Only Does Not Mean File Permissions

It is important to distinguish a read-only filesystem from normal Unix file permissions.

For example:

```text
-r--r--r--
```

This means that the permissions of a particular file only allow reading.

These permissions may normally be changed with commands such as `chmod`.

A read-only filesystem is different.

The entire mounted filesystem refuses operations that modify its contents.

For example, operations such as the following would fail when performed directly on a read-only filesystem:

```sh
touch file
rm file
mkdir directory
echo "hello" > file
```

The kernel typically reports this as:

```text
Read-only file system
```

In Linux, the corresponding error is `EROFS`.

So there are two separate concepts:

```text
File permissions
    │
    └── Who is allowed to read, write, or execute this file?

Filesystem mount mode
    │
    └── Does this filesystem allow write operations at all?
```

Even if a file has writable permissions, a filesystem mounted as read-only can still prevent it from being modified.

### Why Is This Useful?

A read-only base filesystem protects the original operating-system files from modification.

Linux Live systems commonly combine a read-only base filesystem with a writable layer:

```text
             Root filesystem (/)
                    │
            ┌───────┴───────┐
            │               │
      Writable layer   Read-only layer
         changes            /rofs
                              │
                        original files
```

When nothing has been changed, the file visible through `/` may simply come from the read-only layer.

This explains why `/usr/sbin/john` and `/rofs/usr/sbin/john` contain exactly the same data in this level.

This filesystem design is closely related to concepts such as:

- Live Linux systems
- OverlayFS / AUFS
- Copy-on-Write (CoW)
- Container image layers
- Immutable operating systems

We will encounter similar ideas later in Linux systems and container technologies such as Docker.

Here is the command to check it:

```sh
cat /usr/sbin/john | tr 'A-Za-z' 'L-ZA-KI-za-k'
```

We have to understand the Caesar cipher to know why the line looks like this.

So, what is a Caesar cipher? It is a classic substitution cipher that shifts the alphabet by a specific amount.

When encrypting:

- `E(x) = (x + k) mod 26` (`k` is the shift)

When decrypting:

- `E(x) = (x - k) mod 26` (`k` is the shift)

Why is the Caesar cipher insecure?

As with our command line, we can break it by brute force because it has only 25 possible non-zero shifts.
