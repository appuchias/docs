---
created: 2023-09-30T12:48:07 (UTC +02:00)
tags:
  - hardware
  - server
  - Ubuntu
  - power
source: https://askubuntu.com/questions/39760/how-can-i-control-hdd-spin-down-time
author:
---

# hard drive - How can I control HDD spin down time? - Ask Ubuntu

> ## Excerpt
> I have 2 HDDs in my PC. Ubuntu is turning off the secondary HDD very quickly after about 15 minutes, which is short for me. I need to control this time. How can I do it?

Have a look at `hdparm`.

From [the manual](http://manpages.ubuntu.com/manpages/quantal/en/man8/hdparm.8.html) (`man hdparm` on the command line):

```
-S     Put the drive into idle (low-power) mode,  and  also  set  the  standby  (spindown)
       timeout  for  the  drive.  This timeout value is used by the drive to determine how
       long to wait (with no disk activity) before turning off the spindle motor  to  save
       power.   Under  such  circumstances,  the  drive  may take as long as 30 seconds to
       respond to a subsequent disk access, though most  drives  are  much  quicker.   The
       encoding  of  the  timeout  value  is  somewhat  peculiar.   A  value of zero means
       "timeouts are disabled": the device will  not  automatically  enter  standby  mode.
       Values  from  1  to  240  specify  multiples of 5 seconds, yielding timeouts from 5
       seconds to 20 minutes.  Values from 241 to 251 specify from 1 to  11  units  of  30
       minutes,  yielding timeouts from 30 minutes to 5.5 hours.  A value of 252 signifies
       a timeout of 21 minutes. A value  of  253  sets  a  vendor-defined  timeout  period
       between  8  and  12 hours, and the value 254 is reserved.  255 is interpreted as 21
       minutes plus 15 seconds.  Note that some  older  drives  may  have  very  different
       interpretations of these values.
```

So:

```
sudo hdparm -I /dev/sdb | grep level
```

will show the current spindown value, for example:

```
Advanced power management level: 254
```

From the manual, 254 is reserved, so I expect it to be Ubuntu's default (can anyone confirm/expand on this please?).

Example:

- `sudo hdparm -S 25 /dev/sdb` = spindown after 25*5 seconds.
    
- `sudo hdparm -S 245 /dev/sdb` = spindown after (245-240)*30 minutes.