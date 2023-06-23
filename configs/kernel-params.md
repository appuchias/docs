Disabling intel integrated graphics made tty reappear.

Added boot parameter `ibt=off` to fix VirtualBox startup issues.

To disable Intel graphics driver, create `/etc/modprobe.d/disable-intel-graphics.conf` and write:
```SH
options i915 modeset=0
```

Setting the file allowed my system to be suspended after some weeks of problems.
