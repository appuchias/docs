---
created: 2023-12-19T19:11:52 (UTC +01:00)
tags: []
source: https://wiki.archlinux.org/title/Fprint#Login_configuration
author: 
---

# fprint - ArchWiki

> ## Excerpt
> From the fprint homepage:

---
From [the fprint homepage](https://www.freedesktop.org/wiki/Software/fprint/):

The fprint project aims to plug a gap in the Linux desktop: support for consumer fingerprint reader devices.

The idea is to use the built-in fingerprint reader in some notebooks for login using [PAM](https://wiki.archlinux.org/title/PAM "PAM"). This article will also explain how to use regular password for backup login method (solely fingerprint scanner is not recommended due to numerous reasons).

## Prerequisites

Make sure you have one of the supported finger scanners. You can check if your device is supported by checking the [list of supported devices](https://fprint.freedesktop.org/supported-devices.html) or the [list of unsupported devices](https://gitlab.freedesktop.org/libfprint/wiki/-/wikis/Unsupported-Devices). To check which one you have, type:

```
$ lsusb
```

The _lsusb_ tool is available inside the [usbutils](https://archlinux.org/packages/?name=usbutils) package.

**Note:** The above list of supported devices is not updated regularly and is not complete. It is worth testing your device using the instructions on this page even if it does not appear on that list, prior to resorting to [AUR](https://wiki.archlinux.org/title/AUR "AUR") packages.

## Installation

[Install](https://wiki.archlinux.org/title/Install "Install") the [fprintd](https://archlinux.org/packages/?name=fprintd) package. [imagemagick](https://archlinux.org/packages/?name=imagemagick) might also be needed.

Some devices require a different fork of [libfprint](https://archlinux.org/packages/?name=libfprint), not (yet?) merged with the main _libfprint_:

-   **libfprint-tod** — For touch-based sensors.

[https://gitlab.freedesktop.org/3v1n0/libfprint/-/tree/tod](https://gitlab.freedesktop.org/3v1n0/libfprint/-/tree/tod) || [libfprint-tod-git](https://aur.archlinux.org/packages/libfprint-tod-git/)<sup><small>AUR</small></sup>

-   **libfprint-elanmoc2** — For ELAN `04f3:0c4c` waiting [merge](https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/330).

[https://gitlab.freedesktop.org/Depau/libfprint/-/tree/elanmoc2](https://gitlab.freedesktop.org/Depau/libfprint/-/tree/elanmoc2) || [libfprint-elanmoc2-git](https://aur.archlinux.org/packages/libfprint-elanmoc2-git/)<sup><small>AUR</small></sup>

-   **libfprint-elanmoc2-newdrvs** — **Experimental** for `04f3:0c4c` or a `04f3:0c00`, waiting [merge](https://gitlab.freedesktop.org/Depau/libfprint/-/merge_requests/1).

[https://gitlab.freedesktop.org/geodic/libfprint/-/tree/elanmoc2](https://gitlab.freedesktop.org/geodic/libfprint/-/tree/elanmoc2) || [libfprint-elanmoc2-newdrvs-git](https://aur.archlinux.org/packages/libfprint-elanmoc2-newdrvs-git/)<sup><small>AUR</small></sup>

## Configuration

### Login configuration

**Note:**

-   If you use [GDM](https://wiki.archlinux.org/title/GDM "GDM"), the fingerprint-option is already available in the login menu (if not add yourself to the `input` [user group](https://wiki.archlinux.org/title/User_group "User group")). You can skip this section!
-   If you use [SDDM](https://wiki.archlinux.org/title/SDDM "SDDM"), see [SDDM#Using a fingerprint reader](https://wiki.archlinux.org/title/SDDM#Using_a_fingerprint_reader "SDDM").

Add `pam_fprintd.so` as _sufficient_ to the top of the auth section of `/etc/pam.d/system-local-login`:

```
/etc/pam.d/system-local-login
```

```
<b>auth      sufficient pam_fprintd.so</b>
auth      include   system-login
...
```

This tries to use fingerprint login first, and if it fails or if it finds no fingerprint signatures in the given user's home directory, it proceeds to password login.

You can also modify other files in `/etc/pam.d/{login,su,sudo,gdm,lightdm}` in the same way. For example `/etc/pam.d/polkit-1` for [polkit](https://wiki.archlinux.org/title/Polkit "Polkit") based authentication (GNOME & many other desktop environments) or `/etc/pam.d/kde` for unlocking KDE's lockscreen.

Adding `pam_fprintd.so` as _sufficient_ to any configuration file in `/etc/pam.d/` when a fingerprint signature is present will only prompt for fingerprint authentication. This prevents the use of a password if you cannot `Ctrl+c` fingerprint authentication (due to the lack of a shell). In order to use either a password or a fingerprint in a graphical interface, add the following line to the top of any files required:

```
<b>authsufficient  pam_unix.so try_first_pass likeauth nullok</b>
authsufficient  pam_fprintd.so
...
```

This will prompt for a password; pressing `Enter` on a blank field will proceed to fingerprint authentication.

If you want to prompt for fingerprint and password input at the same time, you can use [pam-fprint-grosshack](https://aur.archlinux.org/packages/pam-fprint-grosshack/)<sup><small>AUR</small></sup>. This may be needed for some graphical programs which do not allow blank password input, such as Gnome's built-in polkit agent. To use this package, add the following lines to the top of any files required:

```
<b>authsufficient  pam_fprintd_grosshack.so</b>
<b>authsufficient  pam_unix.so try_first_pass nullok</b>
...
```

### Create fingerprint signature

You will need to have an [authentication agent](https://wiki.archlinux.org/title/Polkit#Authentication_agents "Polkit") running before being able to enroll.

To add a signature for a finger, run:

```
$ fprintd-enroll
```

or create a new signature for all fingers:

```
$ fprintd-delete "$USER"
$ for finger in {left,right}-{thumb,{index,middle,ring,little}-finger}; do fprintd-enroll -f "$finger" "$USER"; done
```

You will be asked to scan the given finger. Swipe your right index finger **five times**. After that, the signature is created in `/var/lib/fprint/`.

You can also enroll without an authentication agent:

```
# fprintd-enroll <i>user</i>
```

To verify the newly created fingerprint, use:

```
$ fprintd-verify
```

For more information, see [fprintd(1)](https://man.archlinux.org/man/fprintd.1).

### Restrict enrolling

[![](https://wiki.archlinux.org/images/7/77/Merge-arrows-2.svg)](https://wiki.archlinux.org/title/File:Merge-arrows-2.svg)**This article or section is a candidate for merging with [polkit](https://wiki.archlinux.org/title/Polkit "Polkit").**[![](https://wiki.archlinux.org/images/7/77/Merge-arrows-2.svg)](https://wiki.archlinux.org/title/File:Merge-arrows-2.svg)

**Notes:** Partially duplicates article (Discuss in [Talk:Fprint](https://wiki.archlinux.org/title/Talk:Fprint))

By default every user is allowed to enroll new fingerprints without prompting for the password or the fingerprint. You can change this behavior using [polkit](https://wiki.archlinux.org/title/Polkit "Polkit") rules.

There are two locations that contains the polkit configuration files:

-   `/etc/polkit-1/rules.d/`
-   `/usr/share/polkit-1/rules.d/`

**Note:** You should **not** modify the files under `/usr/share/polkit-1/rules.d/` because they will be overwritten on update. Copy them to `/etc/polkit-1/rules.d/` first.

In the following example only root can enroll fingerprints:

```
/etc/polkit-1/rules.d/50-net.reactivated.fprint.device.enroll.rules
```

```
polkit.addRule(function (action, subject) {
  if (action.id == "net.reactivated.fprint.device.enroll") {
    return subject.user == "root"&nbsp;? polkit.Result.YES&nbsp;: polkit.Result.NO
  }
})
```

## Troubleshooting

### No devices available

If your [supported device](https://fprint.freedesktop.org/supported-devices.html) cannot be found or is claimed to be already open (in use), check the `fprintd.service` logs in the [journal](https://wiki.archlinux.org/title/Journal "Journal").

You may find log entries like:

```
fprintd[2936592]: Corrupted message received
fprintd[2936592]: Ignoring device due to initialization error: unsupported firmware version
```

Ensure your device's firmware is up to date with [Fwupd](https://wiki.archlinux.org/title/Fwupd "Fwupd").

### gdm hangs when revealing login prompt after suspend

This issue is described [in libfprint repository](https://gitlab.freedesktop.org/libfprint/libfprint/-/issues/426). The developers answer is:

My guess right now is that we are disconnecting the BT USB dongle while it is being initialised. Then everything gets stuck while btusb is trying to load the firmware (this has a 10s timeout, explaining the just under 10s hang that we are seeing). Disconnecting the bluetooth dongle like this is expected to happen when the rfkill switch is toggled, so that is normal. It just seems that the case where the device suddenly disconnects is not handled properly and times out.

The proposed fix is to [create](https://wiki.archlinux.org/title/Create "Create"):

```
/etc/modprobe.d/bluetooth-blacklist.conf
```

```
blacklist btusb
```

Or execute straight away:

```
# rmmod btusb
```

Then it should not try to initialize the device.

### Unexpected error while suspending device

This issue is described [in libfprint repository](https://gitlab.freedesktop.org/libfprint/libfprint/-/issues/538):

You need to set your laptop to not suspend to RAM but to do s2idle. You might need to switch the BIOS into "Windows mode".

### Debug

```
# G_MESSAGES_DEBUG=all /usr/lib/fprintd -t
```
