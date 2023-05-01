```{=mediawiki}
{{Lowercase title}}
```
[de:Systemd-boot](de:Systemd-boot "wikilink")
[es:Systemd-boot](es:Systemd-boot "wikilink")
[ja:Systemd-boot](ja:Systemd-boot "wikilink")
[pt:Systemd-boot](pt:Systemd-boot "wikilink")
[ru:Systemd-boot](ru:Systemd-boot "wikilink")
[zh-hans:Systemd-boot](zh-hans:Systemd-boot "wikilink")
`{{Related articles start}}`{=mediawiki}
`{{Related|Arch boot process}}`{=mediawiki}
`{{Related|Secure Boot}}`{=mediawiki}
`{{Related|Unified Extensible Firmware Interface}}`{=mediawiki}
`{{Related articles end}}`{=mediawiki}

**systemd-boot**, previously called **gummiboot** (German for \"rubber
dinghy\"), is an easy-to-configure [UEFI](UEFI "wikilink") [boot
manager](boot_manager "wikilink"). It provides a textual menu to select
the boot entry and an editor for the kernel command line. It is included
with `{{Pkg|systemd}}`{=mediawiki}.

Note that *systemd-boot* can only start EFI executables (e.g., the Linux
kernel [EFISTUB](EFISTUB "wikilink"), [UEFI
shell](UEFI_shell "wikilink"), [GRUB](GRUB "wikilink"), or the [Windows
Boot
Manager](https://learn.microsoft.com/en-us/windows-hardware/drivers/bringup/boot-and-uefi#understanding-the-windows-boot-manager)).

## Installation

### Installing the EFI boot manager {#installing_the_efi_boot_manager}

To install *systemd-boot*, first make sure that the system is booted
into UEFI mode and [UEFI
variables](Unified_Extensible_Firmware_Interface#UEFI_variables "wikilink")
are accessible. This can be verified by running
`{{ic|efivar --list}}`{=mediawiki} or, if `{{Pkg|efivar}}`{=mediawiki}
is not installed, by running
`{{ic|ls /sys/firmware/efi/efivars}}`{=mediawiki} (if the directory
exists, the system is booted into UEFI mode.)

Throughout, `{{ic|''esp''}}`{=mediawiki} will denote the [ESP
mountpoint](EFI_system_partition#Typical_mount_points "wikilink"), e.g.
`{{ic|/efi}}`{=mediawiki} or `{{ic|/boot}}`{=mediawiki}. This assumes
that you have [chrooted](chroot "wikilink") to the system\'s mount
point.

Use `{{man|1|bootctl}}`{=mediawiki} to install *systemd-boot* to the
ESP:

`# bootctl install`

This will copy the *systemd-boot* EFI boot manager to the ESP: on an x64
architecture system
`{{ic|/usr/lib/systemd/boot/efi/systemd-bootx64.efi}}`{=mediawiki} will
be copied to
`{{ic|''esp''/EFI/systemd/systemd-bootx64.efi}}`{=mediawiki} and
`{{ic|''esp''/EFI/BOOT/BOOTX64.EFI}}`{=mediawiki}, and *systemd-boot*
will be set as the default EFI application.

```{=mediawiki}
{{Note|
* When running {{ic|bootctl install}}, ''systemd-boot'' will try to locate the ESP at {{ic|/efi}}, {{ic|/boot}}, and {{ic|/boot/efi}}. Setting {{ic|''esp''}} to a different location requires passing the {{ic|1=--esp-path=''esp''}} option. (See {{man|1|bootctl|OPTIONS}} for details.)
* Installing ''systemd-boot'' will overwrite any existing {{ic|''esp''/EFI/BOOT/BOOTX64.EFI}}, e.g. Microsoft's version of the file.
}}
```
To conclude the installation, [configure](#Configuration "wikilink")
*systemd-boot*.

### Installation using XBOOTLDR {#installation_using_xbootldr}

A separate [/boot partition](Partitioning#/boot "wikilink") of type
\"Linux extended boot\" (XBOOTLDR) can be created to keep the kernel and
initramfs separate from the ESP. This is particularly helpful to [dual
boot with Windows](dual_boot_with_Windows "wikilink") with an existing
ESP that is too small.

Prepare an ESP as usual and create another partition for XBOOTLDR on the
same physical drive. The XBOOTLDR partition must have a partition type
GUID of `{{ic|bc13c2ff-59e6-4262-a352-b275fd6f7172}}`{=mediawiki}
[1](https://uapi-group.org/specifications/specs/boot_loader_specification/).
The size of the XBOOTLDR partition should be large enough to accommodate
all of the kernels you are going to install.

```{=mediawiki}
{{Note|
* ''systemd-boot'' does not do a file system check like it does for the ESP. Hence, it is possible to use any file system that your UEFI implementation can read.
* UEFI may skip loading partitions other than the ESP when a "fast boot" mode is enabled. This can lead to ''systemd-boot'' failing to find entries on the XBOOTLDR partition; in that case, disable the "fast boot" mode.
* The XBOOTLDR partition must be on the same physical disk as the ESP for ''systemd-boot'' to recognize it.
}}
```
During install, mount the ESP to `{{ic|/mnt/efi}}`{=mediawiki} and the
XBOOTLDR partition to `{{ic|/mnt/boot}}`{=mediawiki}.

Once in chroot, use the command:

`# bootctl --esp-path=/efi --boot-path=/boot install`

To conclude the installation, [configure](#Configuration "wikilink")
*systemd-boot*.

### Updating the EFI boot manager {#updating_the_efi_boot_manager}

Whenever there is a new version of *systemd-boot*, the EFI boot manager
can be optionally reinstalled by the user. This can be done manually or
automatically; the two approaches are described thereafter.

```{=mediawiki}
{{Note|The EFI boot manager is a standalone EFI executable and any version can be used to boot the system (partial updates do not apply, since pacman only installs the ''systemd-boot'' installer, not ''systemd-boot'' itself.) However, new versions may add new features or fix bugs, so it is probably a good idea to update ''systemd-boot''.}}
```
#### Manual update {#manual_update}

Use *bootctl* to update *systemd-boot*:

`# bootctl update`

```{=mediawiki}
{{Note|As with {{ic|bootctl install}}, ''systemd-boot'' will try to locate the ESP at {{ic|/efi}}, {{ic|/boot}}, and {{ic|/boot/efi}}. Setting {{ic|''esp''}} to a different location requires passing the {{ic|1=--esp-path=''esp''}} option.}}
```
#### Automatic update {#automatic_update}

To update *systemd-boot* automatically, either use a [systemd
service](Systemd#Using_units "wikilink") or a [pacman
hook](pacman_hook "wikilink"). The two methods are described below.

##### systemd service {#systemd_service}

As of version 250, `{{Pkg|systemd}}`{=mediawiki} ships with
`{{ic|systemd-boot-update.service}}`{=mediawiki}.
[Enabling](Enabling "wikilink") this service will update the bootloader
upon the next boot.

```{=mediawiki}
{{Warning|If you have [[Secure Boot]] enabled, you need to sign your bootloader after updating. See [[#pacman hook]] below for an example of how to do so.}}
```
```{=mediawiki}
{{Tip|{{ic|bootctl install}} and {{ic|bootctl update}} search {{ic|.efi.signed}} files under {{ic|/usr/lib/systemd/boot/efi/}} before {{ic|.efi}} files. A {{ic|.efi.signed}} bootloader under {{ic|/usr/lib/systemd/boot/efi/}} allows {{ic|systemd-boot-update.service}} to update the bootloader without invoking Secure Boot signing tools. (See {{man|1|bootctl|SIGNED .EFI FILES}} for details.)}}
```
##### pacman hook {#pacman_hook}

The package `{{AUR|systemd-boot-pacman-hook}}`{=mediawiki} adds a pacman
hook which is executed every time `{{Pkg|systemd}}`{=mediawiki} is
upgraded.

Rather than installing *systemd-boot-pacman-hook*, you may prefer to
manually place the following file in
`{{ic|/etc/pacman.d/hooks/}}`{=mediawiki}:

```{=mediawiki}
{{hc|/etc/pacman.d/hooks/95-systemd-boot.hook|2=
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Gracefully upgrading systemd-boot...
When = PostTransaction
Exec = /usr/bin/systemctl restart systemd-boot-update.service
}}
```
If you have [Secure Boot](Secure_Boot "wikilink") enabled, you may want
to add a pacman hook to automatically re-sign the kernel and bootloader
upon every upgrade of the respective packages:

```{=mediawiki}
{{hc|/etc/pacman.d/hooks/99-secureboot.hook|2=
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux
Target = systemd

[Action]
Description = Signing Kernel for Secure Boot
When = PostTransaction
Exec = /usr/bin/find /boot -type f ( -name vmlinuz-* -o -name systemd* ) -exec /usr/bin/sh -c 'if ! /usr/bin/sbverify --list {} 2>/dev/null {{!}}
```
/usr/bin/grep -q \"signature certificates\"; then /usr/bin/sbsign
**\--key *keyfile* \--cert *certfile*** \--output \"\$1\" \"\$1\"; fi\'
\_ {} ; Depends = sbsigntools Depends = findutils Depends = grep }}

Make sure that the parameters
`{{ic|--key ''keyfile'' --cert ''certfile''}}`{=mediawiki} point to your
signing key and certificate. For better understanding of this hook,
consult `{{man|1|sbverify}}`{=mediawiki} and
`{{man|1|sbsign}}`{=mediawiki}.

```{=mediawiki}
{{Tip|If you are using {{Pkg|sbctl}} then resigning of the files enrolled in its database is automatically done through the hook provided at {{ic|/usr/share/libalpm/hooks/zz-sbctl.hook}}. Remember to enroll the files necessary to your bootchain first.}}
```
## Configuration

### Loader configuration {#loader_configuration}

The loader configuration is stored in the file
`{{ic|''esp''/loader/loader.conf}}`{=mediawiki}. See
`{{man|5|loader.conf|OPTIONS}}`{=mediawiki} for details.

A loader configuration example is provided below:

```{=mediawiki}
{{hc|''esp''/loader/loader.conf|
default  arch.conf
timeout  4
console-mode max
editor   no
}}
```
```{=mediawiki}
{{Tip|
* systemd-boot does not accept tabs for indentation, use spaces instead.
* {{ic|default}} and {{ic|timeout}} can be changed in the boot menu itself and changes will be stored as EFI variables {{ic|LoaderEntryDefault}} and {{ic|LoaderConfigTimeout}}, overriding these options. 
* {{ic|bootctl set-default ""}} and {{ic|bootctl set-timeout ""}} can be used to clear the EFI variables overriding the {{ic|default}} and {{ic|timeout}} options, respectively.
* If you have set {{ic|timeout 0}}, the boot menu can be accessed by pressing {{ic|Space}}.
* A basic loader configuration file is located at {{ic|/usr/share/systemd/bootctl/loader.conf}}.
* If the bootloader (during the entry selection) appears distorted/uses the wrong resolution you can try to set the {{ic|console-mode}} to {{ic|auto}} (uses heuristics to select the best resolution), {{ic|keep}} (keeps the firmware provided resolution) or {{ic|2}} (tries to select the first non-UEFI-standard resolution).
}}
```
### Adding loaders {#adding_loaders}

*systemd-boot* will search for boot menu items in
`{{ic|''esp''/loader/entries/*.conf}}`{=mediawiki} and additionally in
`{{ic|''boot''/loader/entries/*.conf}}`{=mediawiki} if using
[XBOOTLDR](#Installation_using_XBOOTLDR "wikilink"). Note that entries
in `{{ic|''esp''}}`{=mediawiki} can only use files (e.g. kernels,
initramfs, images, etc.) in `{{ic|''esp''}}`{=mediawiki}. Similarly,
entries in `{{ic|''boot''}}`{=mediawiki} can only use files in
`{{ic|''boot''}}`{=mediawiki}.

```{=mediawiki}
{{Remove|Duplicates https://uapi-group.org/specifications/specs/boot_loader_specification/#type-1-boot-loader-specification-entries.|section=Potential removal of "Loader configuration" section}}
```
The possible options are:

-   ```{=mediawiki}
    {{ic|title}}
    ```
    -- operating system name. **Required.**

-   ```{=mediawiki}
    {{ic|version}}
    ```
    -- kernel version, shown only when multiple entries with same title
    exist. Optional.

-   ```{=mediawiki}
    {{ic|machine-id}}
    ```
    -- machine identifier from `{{ic|/etc/machine-id}}`{=mediawiki},
    shown only when multiple entries with same title and version exist.
    Optional.

-   ```{=mediawiki}
    {{ic|efi}}
    ```
    -- EFI program to start, relative to your ESP
    (`{{ic|''esp''}}`{=mediawiki}); e.g.
    `{{ic|/vmlinuz-linux}}`{=mediawiki}. **Either** this parameter
    **or** `{{ic|linux}}`{=mediawiki} (see below) **is required**.

-   ```{=mediawiki}
    {{ic|options}}
    ```
    -- space-separated command line options to pass to the EFI program
    or [kernel parameters](kernel_parameters "wikilink"). Optional, but
    you will need at least `{{ic|1=root=''dev''}}`{=mediawiki} if
    booting Linux. This parameter can be omitted if the root partition
    is assigned the correct Root Partition Type GUID as defined in
    [Discoverable Partitions
    Specification](https://uapi-group.org/specifications/specs/discoverable_partitions_specification/)
    and if the `{{ic|systemd}}`{=mediawiki}
    [mkinitcpio](mkinitcpio "wikilink") hook is present.

For Linux boot, you can also use `{{ic|linux}}`{=mediawiki} instead of
`{{ic|efi}}`{=mediawiki}. Or `{{ic|initrd}}`{=mediawiki} in addition to
`{{ic|options}}`{=mediawiki}. The syntax is:

-   ```{=mediawiki}
    {{ic|linux}}
    ```
    or `{{ic|initrd}}`{=mediawiki} followed by the relative path of the
    corresponding files in the ESP; e.g.
    `{{ic|/vmlinuz-linux}}`{=mediawiki}; this will be automatically
    translated into `{{ic|efi ''path''}}`{=mediawiki} and
    `{{ic|1=options initrd=''path''}}`{=mediawiki} -- this syntax is
    only supported for convenience and has no differences in function.

```{=mediawiki}
{{Note|If {{ic|options}} is present in a boot entry and [[Secure Boot]] is disabled, the value of {{ic|options}} will override any {{ic|.cmdline}} string embedded in the EFI image that is specified by {{ic|efi}} or {{ic|linux}} (see [[Unified kernel image#Preparing a unified kernel image]]). With Secure Boot, however, {{ic|options}} (and any edits made to the kernel command line in the bootloader UI) will be ignored, and only the embedded {{ic|.cmdline}} will be used. }}
```
An example of loader files launching Arch from a volume
[labeled](Persistent_block_device_naming#by-label "wikilink")
`{{ic|''arch_os''}}`{=mediawiki} and loading Intel CPU
[microcode](microcode "wikilink") is:

```{=mediawiki}
{{hc|''esp''/loader/entries/arch.conf|2=
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root="LABEL=''arch_os''" rw
}}
```
```{=mediawiki}
{{hc|''esp''/loader/entries/arch-fallback.conf|2=
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux-fallback.img
options root="LABEL=''arch_os''" rw
}}
```
*systemd-boot* will automatically check at boot time for **Windows Boot
Manager** at the location
`{{ic|/EFI/Microsoft/Boot/Bootmgfw.efi}}`{=mediawiki}, [UEFI
shell](UEFI_shell "wikilink") `{{ic|/shellx64.efi}}`{=mediawiki} and
**EFI Default Loader** `{{ic|/EFI/BOOT/bootx64.efi}}`{=mediawiki}, as
well as specially prepared kernel files found in
`{{ic|/EFI/Linux/}}`{=mediawiki}. When detected, corresponding entries
with titles `{{ic|auto-windows}}`{=mediawiki},
`{{ic|auto-efi-shell}}`{=mediawiki} and
`{{ic|auto-efi-default}}`{=mediawiki}, respectively, will be generated.
These entries do not require manual loader configuration. However, it
does not auto-detect other EFI applications (unlike
[rEFInd](rEFInd "wikilink")), so for booting the Linux kernel, manual
configuration entries must be created.

```{=mediawiki}
{{Tip|
* The available boot entries which have been configured can be listed with the command {{ic|bootctl list}}.
* An example entry file is located at {{ic|/usr/share/systemd/bootctl/arch.conf}}.
* The [[kernel parameters]] for scenarios such as [[LVM]], [[LUKS]] or [[dm-crypt]] can be found on the relevant pages.
}}
```
#### EFI Shells or other EFI applications {#efi_shells_or_other_efi_applications}

In case you installed an [EFI shell](UEFI_shell "wikilink") with the
package `{{Pkg|edk2-shell}}`{=mediawiki}, *systemd-boot* will
auto-detect and create a new entry if the EFI file is placed in
`{{ic|''esp''/shellx64.efi}}`{=mediawiki}. To perform this and example
command after installing the package would be:

`# cp /usr/share/edk2-shell/x64/Shell.efi /boot/shellx64.efi`

Otherwise in case you installed [other EFI
applications](rEFInd#Tools "wikilink") into the ESP, you can use the
following snippets.

```{=mediawiki}
{{Note|The file path parameter for the {{ic|efi}} line is relative to your ''esp'' mount point. If you are mounted on {{ic|/boot}} and your EFI binaries reside at {{ic|/boot/EFI/xx.efi}} and {{ic|/boot/yy.efi}}, then you would specify the parameters as {{ic|efi /EFI/xx.efi}} and {{ic|efi /yy.efi}} respectively.}}
```
```{=mediawiki}
{{hc|''esp''/loader/entries/fwupd.conf|
title  Firmware updator
efi     /EFI/tools/fwupdx64.efi
}}
```
```{=mediawiki}
{{hc|''esp''/loader/entries/gdisk.conf|
title  GPT fdisk (gdisk)
efi     /EFI/tools/gdisk_x64.efi
}}
```
#### Boot from another disk {#boot_from_another_disk}

*systemd-boot* cannot launch binaries from partitions other than the ESP
or the XBOOTLDR partition, but it can run an external script to do so.

First we need to install `{{Pkg|edk2-shell}}`{=mediawiki} (will be the
interpreter to be used) and using the EFI shell (as explained above) we
can use the *map* command to take notes of the **FS alias** (ex:
HD0a66666a2) and the full **path** of the destination EFI file (ex:
EFI\\Microsoft\\Boot\\Bootmgfw.efi).

Then using the *exit* command we can boot back into Linux where we can
create the new entry. To do so we need to first create in the root of
the *esp* mount point a *.nsh* filename with the FS alias, a colon and
the EFI path, here an example:

```{=mediawiki}
{{hc|''esp''/windows.nsh|
HD0a66666a2:EFI\Microsoft\Boot\Bootmgfw.efi
}}
```
Once we created this file we can proceed to create the loader entry to
run the script:

```{=mediawiki}
{{hc|''esp''/loader/entries/windows.conf|
title  Windows
efi     /shellx64.efi
options -nointerrupt -noconsolein -noconsoleout windows.nsh
}}
```
Its important that the *efi* path matches where the edk2-shell has been
copied in the *esp* partition, and the last argument of options matches
the *.nsh* filename in the root of the *esp* partition. Also note that
the edk2-shell EFI file can be moved to avoid the entry auto-creation of
*systemd-boot*.

### Booting into EFI Firmware Setup {#booting_into_efi_firmware_setup}

systemd-boot will automatically add an entry to boot into UEFI Firmware
Setup if your device\'s firmware supports rebooting into setup from the
OS.

### Support hibernation {#support_hibernation}

See [Suspend and hibernate](Suspend_and_hibernate "wikilink").

### Kernel parameters editor with password protection {#kernel_parameters_editor_with_password_protection}

Alternatively you can install
`{{AUR|systemd-boot-password}}`{=mediawiki} which supports
`{{ic|password}}`{=mediawiki} basic configuration option. Use
`{{ic|sbpctl generate}}`{=mediawiki} to generate a value for this
option.

Install *systemd-boot-password* with the following command:

`# sbpctl install `*`esp`*

With enabled editor you will be prompted for your password before you
can edit kernel parameters.

## Tips and tricks {#tips_and_tricks}

### Keys inside the boot menu {#keys_inside_the_boot_menu}

See `{{man|7|systemd-boot|KEY BINDINGS}}`{=mediawiki} for the available
key bindings inside the boot menu.

### Choosing next boot {#choosing_next_boot}

The boot manager is integrated with the systemctl command, allowing you
to choose what option you want to boot after a reboot. For example,
suppose you have built a custom kernel and created an entry file
`{{ic|''esp''/loader/entries/arch-custom.conf}}`{=mediawiki} to boot
into it, you can just launch

`$ systemctl reboot --boot-loader-entry=arch-custom.conf`

and your system will reboot into that entry maintaining the default
option intact for subsequent boots. To see a list of possible entries
pass the `{{ic|1=--boot-loader-entry=help}}`{=mediawiki} option.

If you want to boot into the firmware of your motherboard directly, then
you can use this command:

`$ systemctl reboot --firmware-setup`

### Unified kernel images {#unified_kernel_images}

[Unified kernel images](Unified_kernel_image "wikilink") in
`{{ic|''esp''/EFI/Linux/}}`{=mediawiki} are automatically sourced by
systemd-boot, and do not need an entry in
`{{ic|''esp''/loader/entries}}`{=mediawiki}. (Note that unified kernel
images must have a `{{ic|.efi}}`{=mediawiki} extension to be identified
by *systemd-boot*.)

```{=mediawiki}
{{Tip|Files in {{ic|''esp''/loader/entries}} will be booted first if no {{ic|default}} is set in {{ic|''esp''/loader/loader.conf}}. Remove those entries, or set the default with the full file name, ie {{ic|1=default archlinux-linux.efi}}}}
```
### Grml on ESP {#grml_on_esp}

```{=mediawiki}
{{Note|The following instructions are not exclusive to Grml. With slight adjustments, installing other software (e.g., [https://www.system-rescue-cd.org/ SystemRescueCD]) is possible.}}
```
```{=mediawiki}
{{Tip|A {{ic|PKGBUILD}} is available: {{AUR|grml-systemd-boot}}.}}
```
[Grml](https://grml.org/) is a small live system with a collection of
software for system administration and rescue.

In order to install Grml on the ESP, we only need to copy the kernel
`{{ic|vmlinuz}}`{=mediawiki}, the initramfs
`{{ic|initrd.img}}`{=mediawiki}, and the squashed image
`{{ic|grml64-small.squashfs}}`{=mediawiki} from the iso file to the ESP.
To do so, first download [grml64-small.iso](https://grml.org/) and mount
the file (the mountpoint is henceforth denoted *mnt*); the kernel and
initramfs are located in `{{ic|''mnt''/boot/grml64small/}}`{=mediawiki},
and the squashed image resides in
`{{ic|''mnt''/live/grml64-small/}}`{=mediawiki}.

Next, create a directory for Grml in your ESP,

`# mkdir -p `*`esp`*`/grml`

and copy the above-mentioned files in there:

`# cp `*`mnt`*`/boot/grml64small/vmlinuz `*`esp`*`/grml`\
`# cp `*`mnt`*`/boot/grml64small/initrd.img `*`esp`*`/grml`\
`# cp `*`mnt`*`/live/grml64-small/grml64-small.squashfs `*`esp`*`/grml`

In the last step, create an entry for the systemd-boot loader: In
`{{ic|''esp''/loader/entries}}`{=mediawiki} create a
`{{ic|grml.conf}}`{=mediawiki} file with the following content:

```{=mediawiki}
{{hc|''esp''/loader/entries/grml.conf|2=
title   Grml Live Linux
linux   /grml/vmlinuz
initrd  /grml/initrd.img
options apm=power-off boot=live live-media-path=/grml/ nomce net.ifnames=0
}}
```
For an overview of the available boot options, consult the [cheatcode
for
Grml](https://git.grml.org/?p=grml-live.git;a=blob_plain;f=templates/GRML/grml-cheatcodes.txt;hb=HEAD).

### systemd-boot on BIOS systems {#systemd_boot_on_bios_systems}

If you need a bootloader for BIOS systems that follows [The Boot Loader
Specification](https://uapi-group.org/specifications/specs/boot_loader_specification/),
then systemd-boot can be pressed into service on BIOS systems. The
[Clover](Clover "wikilink") boot loader supports booting from BIOS
systems and provides a simulated EFI environment.

## Troubleshooting

### Installing after booting in BIOS mode {#installing_after_booting_in_bios_mode}

```{=mediawiki}
{{Note|This is not recommended.}}
```
If booted in BIOS mode, you can still install *systemd-boot*, however
this process requires you to tell firmware to launch *systemd-boot*\'s
EFI file at boot, usually via two ways:

-   you have a working EFI Shell somewhere else.
-   your firmware interface provides a way of properly setting the EFI
    file that needs to be loaded at boot time.

If you can do it, the installation is easier: go into your EFI Shell or
your firmware configuration interface and change your machine\'s default
EFI file to
`{{ic|''esp''/EFI/systemd/systemd-bootx64.efi}}`{=mediawiki}.

```{=mediawiki}
{{Note|The firmware interface of Dell Latitude series provides everything you need to setup EFI boot but the EFI Shell will not be able to write to the computer's ROM.}}
```
### Manual entry using efibootmgr {#manual_entry_using_efibootmgr}

If the `{{ic|bootctl install}}`{=mediawiki} command failed, you can
create a EFI boot entry manually using `{{Pkg|efibootmgr}}`{=mediawiki}:

`# efibootmgr --create --disk /dev/sd`*`X`*` --part `*`Y`*` --loader "\EFI\systemd\systemd-bootx64.efi" --label "Linux Boot Manager" --unicode`

where `{{ic|/dev/sd''XY''}}`{=mediawiki} is the [EFI system
partition](EFI_system_partition "wikilink").

```{=mediawiki}
{{Note|The path to the EFI image must use the backslash ({{ic|\}}) as the separator}}
```
### Manual entry using bcdedit from Windows {#manual_entry_using_bcdedit_from_windows}

If for any reason you need to create an EFI boot entry from Windows, you
can use the following commands from an Administrator prompt:

`> bcdedit /copy {bootmgr} /d "Linux Boot Manager"`\
`> bcdedit /set {`*`guid`*`} path \EFI\systemd\systemd-bootx64.efi`

Replace `{{ic|''guid''}}`{=mediawiki} with the id returned by the first
command. You can also set it as the default entry using

`> bcdedit /default {`*`guid`*`}`

### Menu does not appear after Windows upgrade {#menu_does_not_appear_after_windows_upgrade}

See [UEFI#Windows changes boot
order](UEFI#Windows_changes_boot_order "wikilink").

### Add support for Windows BitLocker TPM unlocking {#add_support_for_windows_bitlocker_tpm_unlocking}

To stop BitLocker from requesting the recovery key, add the following to
*loader.conf*:

```{=mediawiki}
{{hc|''esp''/loader/loader.conf|
reboot-for-bitlocker yes
}}
```
This will set the *BootNext* UEFI variable, whereby *Windows Boot
Manager* is loaded without BitLocker requiring the recovery key. This is
a one-time change, and *systemd-boot* remains the default bootloader.
There is no need to specify Windows as an entry if it was autodetected.

This is an experimental feature, so make sure to consult
`{{man|5|loader.conf}}`{=mediawiki}.

## See also {#see_also}

-   <https://www.freedesktop.org/wiki/Software/systemd/systemd-boot/>
-   <https://github.com/systemd/systemd/tree/master/src/boot/efi>
-   <https://bbs.archlinux.org/viewtopic.php?id=254374>
-   <https://uapi-group.org/specifications/specs/boot_loader_specification/>

[Category:Boot loaders](Category:Boot_loaders "wikilink")
