Disabling intel integrated graphics made tty reappear.

Added boot parameter `ibt=off` to fix VirtualBox startup issues.

To disable Intel graphics driver, `blacklist.conf` should be placed in `/etc/modprobe.d/`.
Setting the file allowed my system to be suspended after some weeks of problems.
