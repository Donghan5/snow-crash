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

Sadly, it is not the password. It seems to be encoded.

`/rofs` stands for a read-only filesystem mount in the environment.

In this level, both `/usr/sbin/john` and `/rofs/usr/sbin/john` were found during the search, and both contain the same encoded string.

Here is the command to check it:

```sh
cat /usr/sbin/john | tr 'A-Za-z' 'L-ZA-KI-za-k'
```

Using dCode's Cipher Identifier, the string can be identified as a Caesar cipher. After testing the possible shifts, it produces readable plaintext.

We have to understand the Caesar cipher to know why the `tr` command looks like this.

So, what is a Caesar cipher? It is a classic substitution cipher that shifts the alphabet by a specific amount.

When encrypting:

- `E(x) = (x + k) mod 26` (`k` is the shift)

When decrypting:

- `E(x) = (x - k) mod 26` (`k` is the shift)

Why is the Caesar cipher insecure?

As with our command line, we can break it by brute force because it has only 25 possible non-zero shifts.
