# Find the Flag02 Password

When we log in as `level02`, we can see the `level02.pcap` file. It is a data file containing captured network packets.

As in `level01`, we copy the file from the VM to the local machine for further processing.

## Inspect the Packet Capture

We are going to use Wireshark to view the TCP stream. Alternatively, you can inspect it with the `tshark` command-line tool:

```sh
tshark -r level02.pcap
```

To see more details, use:

```sh
tshark -r level02.pcap -V
```

The `-V` option displays detailed protocol header information for each packet.

If you want to see only TCP packets, use:

```sh
tshark -r level02.pcap -Y tcp
```

The `-Y` option specifies a Wireshark display filter.

## Follow the TCP Stream

To view the TCP stream via the CLI, first check the stream number:

```sh
tshark -r level02.pcap -T fields -e tcp.stream | sort -nu
```

If the result is `0`, it means that there is one TCP conversation.

You can then view the entire TCP stream:

```sh
tshark -r level02.pcap -q -z follow,tcp,ascii,0
```

However, the result may be difficult to read. The following command provides a more compact view:

```sh
tshark -r level02.pcap -T fields -e frame.number -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -e data.data
```

The output looks like this:

```text
1   59.233.235.218   39247   59.233.235.223   12121   50617373776f7264
2   59.233.235.223   12121   59.233.235.218   39247   ...
```

It is still difficult to read, so use:

```sh
tshark -r level02.pcap -T fields -e data.data | tr -d '\n' | xxd -r -p
```

This prints only the payload, without unnecessary headers.

## Interpret the Password

The output appears to contain `Password: ft_wandrNDRelL0L`. Unfortunately, this is not the correct password because the input contains DEL characters.

The original raw bytes are:

```text
66 74 5f 77 61 6e 64 72 7f 7f 7f 4e 44 52 65 6c 7f 4c 30 4c 0d
                        ^^ ^^ ^^             ^^          ^^
                       DEL DEL DEL           DEL         Enter
```

There are four DEL characters in total. Each DEL character removes the character immediately before it:

```text
ft_wandr + DEL + DEL + DEL  -> ft_wa
ft_wa + NDRel + DEL         -> ft_waNDRe
ft_waNDRe + L0L             -> ft_waNDReL0L
```

Therefore, after applying the four DEL inputs, the correct password is:

```text
ft_waNDReL0L
```
