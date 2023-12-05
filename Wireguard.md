After getting config files,
```bash
nmcli connection import type wireguard file <connection>.conf
```
will add it to NetworkManager so it can be controlled from the CLI, TUI, GUI or applets.

However, it will be added with `connection.autoconnect` enabled. Use this to disable it:
```bash
nmcli connection modify <connection> connection.autoconnect no
```

And then, to enable or disable the VPN, use:
```bash
nmcli connection up|down <connection>
```
