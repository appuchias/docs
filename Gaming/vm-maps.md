Increasing vm map removes issues with some games. It may cause issues with old programs trying to access coredumps as they expect a value of 65530 instead of MAX_INT - 5

Path: /etc/sysctl.d/80-gamecompatibility.conf
```SH
vm.max_map_count=2147483642
```

Alternative command: `sudo sysctl -w vm.max_map_count=2147483642 # MAX_INT - 5`

