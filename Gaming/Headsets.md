Linux WIRELESS Gaming Headset: SteelSeries Arctis vs Logitech, and integrating it with the system.
advice wanted

Looking for a wireless gaming headset for Linux. I have found a few ways to get things working on Linux.

I am choosing between either SteelSeries Arctis (Pro Wireless or something like that) or Logitech (G-something). Whatever the top model is of each.

Neither of them natively support Linux and they have major issues such as suddenly turning off while you are using them because their audio-based auto-standby code only works properly on Windows.

But luckily I found a command-line project that can send the correct commands to the headsets to toggle Sidetone (hearing yourself/the room with zero latency), changing the timeout of the auto-standby, etc.

Here is the project: https://github.com/Sapd/HeadsetControl

There is also a little Python appindicator icon that places the headset status and controls in your system tray.

So far it seems like it would work well.

Here is the plan:

- Headset (whichever one has the least hassle on Linux)

- https://github.com/Sapd/HeadsetControl

-  https://github.com/centic9/headset-charge-indicator/ for the tray icon app (battery status and side toggle)

- Easy Effects or whatever the Pipewire thing is called which allows you to place FX in the microphone input chain, to clean up your mic quality, since all these mics rely on Windows software to clean up the mic, so Linux users have to do it ourselves.

 - Finally, the rest of the headset features such as virtual surround definitely won't work on Linux. So that doesn't matter. But the good news is that surround on headphones is a fake scam anyway, since LTT tested both software and hardware based headphone surround and found that all headset manufacturer implementations suck.

- You won't have auto standby, so you have to remember to flip the headset switch when the computer is SILENT to preserve battery. This means that if you like to sit with a headset on even when the computer is silent then your wireless headset will draw a lot more battery than if you used the headset on Windows with proper standby. Edit: With HeadsetControl, you are able to configure the auto-standby timeout and it seems to work properly, just like on Windows.

- And hopefully the headsets remember the auto standby timeout, so that you don't have to constantly toggle it via the tool.

Well, has anyone tried any of this (the wireless Headset control app) or have any advice about wireless headsets that integrate well on Linux? If I have found "the best way" then I guess it doesn't matter which of the two brands I buy. But I am unsure about the whole thing. Seems really hacky to get this to work on Linux and you might have a constantly depleted battery if there are issues with auto standby?

 
Edit: I ended up buying the wired HyperX Cloud Orbit S instead. They don't need charging, they're using USB for audio transfer, and the mic sounds awesome (if you install EasyEffects to boost the gain, do noise reduction and EQ it a bit). The main reasons why I picked it were the fact that it uses audiophile-grade planar magnetic speakers which sound INSANELY good (way better than all other headphones), since they're taken from an actual audiophile brand's headphones. And secondly, because they have Waves NX DSP built-in, which is the world's best virtual surround. When you enable 3D mode, it appears as a 7.1 output on the computer and works perfectly on Linux. I can hear footsteps all around me extremely accurately in games. If I was gonna use a wireless headset in the future, both the Arctis and the Logitech seem like good choices and the "HeadsetControl" CLI tool is able to configure them on Linux. But I wanted something that needs zero configuration in the OS and has perfect 3D virtualiztaion, and for that, the ONLY choice for Linux users (LITERALLY the ONLY headphone in the world) that will do that is the HyperX. And that's because it does the DSP inside hardware without any need for computer software.