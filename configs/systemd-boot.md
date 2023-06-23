Confs in /boot/loader/entries/
Settings in /boot/loader/loader.conf

[Loader settings](systemd-boot-archwiki.md#Configuration)

Updating Pacman hook
Path: /etc/pacman.d/hooks/95-systemd-boot.hook
```SH
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Gracefully upgrading systemd-boot...
When = PostTransaction
Exec = /usr/bin/systemctl restart systemd-boot-update.service
```

Reboot to Windows: `systemctl reboot --boot-loader-entry=auto-windows # ENSURE Windows EFI present`

Add windows option:
**Copy Windows's EFI `Microsoft` folder into `/boot/efi/EFI`**
[source](https://github.com/spxak1/weywot/blob/main/Pop_OS_Dual_Boot.md)
