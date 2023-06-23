Sets the mouse acceleration.

Path:  `/etc/X11/xorg.conf.d/99-mouse.conf`
```SH
Section "InputClass"
  Identifier "10"
  MatchDriver "libinput"
  MatchProduct "Logitech G703"
  Option "AccelSpeed" "-0.8"
EndSection
```
