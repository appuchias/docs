[ArchWiki](https://wiki.archlinux.org/title/Network_configuration#Investigate_sockets)
# Investigate sockets

*ss* is a utility to investigate network ports and is part of the [iproute2](https://archlinux.org/packages/?name=iproute2) package. It has a similar functionality to the [deprecated](https://archlinux.org/news/deprecation-of-net-tools/) netstat utility.

Common usage includes:

Display all TCP Sockets with service names:

```bash
ss -at
```

Display all TCP Sockets with port numbers:

```bash
ss -atn
```

Display all UDP Sockets:

```bash
ss -au
```

For more information see `{{man|8|ss}}`.
