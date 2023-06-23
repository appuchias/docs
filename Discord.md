# Don't auto update

[ArchWiki](https://wiki.archlinux.org/title/Discord#Discord_asks_for_an_update_not_yet_available_in_the_repository)
## Discord asks for an update not yet available in the repository 

Discord will refuse to launch if there is an update available, and the following message will be shown *\"Must be your lucky day, there\'s a new update!\"*. If the updated version is not yet available in the official repos, you can build and install the updated package using the [Arch Build System](Arch_Build_System "wikilink").

To disable the update check, add the following to `~/.config/discord/settings.json`:

```JSON TI:"~/.config/discord/settings.json"
{
	...,
	"SKIP_HOST_UPDATE": true
	...,
}
```

# Start minimized

`--start-minimized` ([source](https://support.discord.com/hc/en-us/community/posts/360042980951-Start-Discord-minimized-on-Linux-provide-a-CLI-switch-for-that))

# Don't die on wake

Discord died when waking up from suspension.

Solutions:
- Disabling Hardware Acceleration: Seems to have fixed it.
- [Preserve NVIDIA VRAM on suspend](https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks#Preserve_video_memory_after_suspend): Currently testing.
	I created `/etc/modprobe.d/nvidia-power-management.conf` containing:
	```SH TI:/etc/modprobe.d/nvidia-power-management.conf
	options nvidia NVreg_PreserveVideoMemoryAllocations=1 NVreg_TemporaryFilePath=/var/tmp
	```
	
	And then [regenerated initramfs](https://wiki.archlinux.org/title/Regenerate_the_initramfs).
