# Killing the Loud Annoying Debian Speaker, Permanently.

I had a humiliating end of the day when my Dell 7577 wouldn't stop beeping, even with sound muted. A few weeks of investigation (there are a lot of idiots in the Linux community) all covered with the deafening high-pitched beeps of the stupid speakers has finally lead me to this.

I decided to permanently kill the speaker drivers.


# How to Fix the Speaker Drivers

1. Try using the often not-working suggested fixes on Google first
2. Then, try my idea, you create a new user, and you change ownership of the drivers from root to newuser
3. Downgrade the permissions for each driver file*.ko (find them with ```updatedb & locate pcspkr```)
4. Downgrade the newuser permissions from 1000 to 9999
5. Remember to deprive the new user of any root permissions or sudo capabilities

That should effectively prevent the bell from ever occuring again. This should work with any Linux distro (Debian based) without SELinux.


# Old Vulgarity Laden Notes

After fruitless methods kept popping up no matter how many times I typed in Google

```
googler 'how the fuck do I uninstall the motherfucking speakers in Debian -windows + -idiots -site:stackoverflow.com -site:books.google.com -site:microsoft.com'
```

[!]()

As it turns out. If options xset or setterm or modprobe blacklist doesnt work, you SHIT OUT OF LUCK. But hey, keep reading. Because apparently this new forced-pcspkr module (as it is called, pcspkr, or in Arch Linux, 'snd_pcsp').

It turns out that modprobe blacklists, is actually deprecated since Kernel 4.9 if my results of ```updatedb && locate pcspkr``` and ```updatedb && locate snd_pcsp```. 

Why the FUCK would you deprecate a command that improves our productivity because we would then DO NOT HAVE BEEPING FUCKING LAPTOPS.

```
/dev/input/by-path/platform-pcspkr-event-spkr
/etc/modprobe.d/pcspkr-blacklist.conf
/lib/modules/4.9.0-4-amd64/kernel/drivers/input/misc/pcspkr.ko
/lib/modules/4.9.0-5-amd64/kernel/drivers/input/misc/pcspkr.ko
/lib/modules/4.9.0-6-amd64/kernel/drivers/input/misc/pcspkr.ko
/root/Pictures/location-of-dumb-assed-pcspkr.png
/run/udev/links/\x2finput\x2fby-path\x2fplatform-pcspkr-event-spkr
/run/udev/links/\x2finput\x2fby-path\x2fplatform-pcspkr-event-spkr/c13:71
/usr/src/linux-headers-4.9.0-4-amd64/include/config/have/pcspkr
/usr/src/linux-headers-4.9.0-4-amd64/include/config/have/pcspkr/platform.h
/usr/src/linux-headers-4.9.0-4-amd64/include/config/input/pcspkr.h
/usr/src/linux-headers-4.9.0-4-amd64/include/config/pcspkr
/usr/src/linux-headers-4.9.0-4-amd64/include/config/pcspkr/platform.h
/usr/src/linux-headers-4.9.0-5-amd64/include/config/have/pcspkr
/usr/src/linux-headers-4.9.0-5-amd64/include/config/have/pcspkr/platform.h
/usr/src/linux-headers-4.9.0-5-amd64/include/config/input/pcspkr.h
/usr/src/linux-headers-4.9.0-5-amd64/include/config/pcspkr
/usr/src/linux-headers-4.9.0-5-amd64/include/config/pcspkr/platform.h
/usr/src/linux-headers-4.9.0-6-amd64/include/config/have/pcspkr
/usr/src/linux-headers-4.9.0-6-amd64/include/config/have/pcspkr/platform.h
/usr/src/linux-headers-4.9.0-6-amd64/include/config/input/pcspkr.h
/usr/src/linux-headers-4.9.0-6-amd64/include/config/pcspkr
/usr/src/linux-headers-4.9.0-6-amd64/include/config/pcspkr/platform.h
root@localhost:~# x2fplatform-pcspkr-event-spkr/c13:71x2fplatform-pcspkr-event-spkr/c13:71x2fplatform-pcspkr-event-spkr/c13:71^C
```

Since this involves me messing with the kernel and the drivers and could potentially cause me to perma-break a stock Debian with a KVM Hypervisor running 11 operating systems at the same time, I will get back to you.

And if this shit fucks up. I will come back even more irate. Meanwhile, you guys, go find the fuck that forced this perma-reinstall so I can punch his nerdy lights out. 

Also, go find the asshole who broke my Intel iwlwifi drivers so I can beat his ass too. Apparently these wifi issues have been lingering for decades, and if I had not learned of penetration testing wireless adapters, I would have a much more bitter look against Linux.

For now, I need to get to fixing this shit using my method. 

# The steps I will take to break the annoying speaker drivers.

# Didnt work, Next One!
XXXX1. Method one, erase the speaker drivers by overwriting the files with zeroes AFTER a apt-get update && apt-get upgrade && apt-get dist upgrade.

The reason  why is that my discovery of the drivers location implies that its being updated with a new, crappier version from each kernel mini update. If you apt-get upgrade at any point from this you might end up reinstalling these annoying drivers.

```apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y && reboot```

After rebooting, do it again. You need to make sure you are at the highest kernel version. Which is somewhere at 4.14 or 4.16.

Now, lets write a script. 

```echo 'updatedb && locate pcspkr && locate snd_pcsp >> shitlist' > find-stupid-drivers.sh```

Run it as ```/bin/sh find-stupid-drivers.sh```

It will generate a file called shitlist.

Now write another script that iterates a file profile for each. We need to get the read write permissions as some of these things were not meant to be edited with. And may break Linux.

```
/lib/modules/4.9.0-4-amd64/kernel/drivers/input/misc/pcspkr.ko
/lib/modules/4.9.0-5-amd64/kernel/drivers/input/misc/pcspkr.ko
/lib/modules/4.9.0-6-amd64/kernel/drivers/input/misc/pcspkr.ko
```

A quick cat-n-grep says, these are the drivers that are responsible. But there is also a symbolic link as well. /dev/input/by-path/platform-pcspkr-event-spkr

And it has a full execution policy. Add that to our list

```
/dev/input/by-path/platform-pcspkr-event-spkr

lXrwxrwxrwx 1 root root 9 Feb 28 00:34 /dev/input/by-path/platform-pcspkr-event-spkr -> ../event7

```
Add all four to another text file called pcspkr-shitlist.txt

Now lets write another script

```
#!/bin/sh
SHELL=/bin/bash

for annoyingbeeps in $(cat pcspkr-shitlist.txt)
	do echo '' > $annoyingbeeps
	chmod 400 $annoyingbeeps
	done
```

We are taking away permissions from the emptied drivers (we totally hosed them by replacing the code with 'nothing') and the dumb beeper-device (thats the one in the /dev/input folder) as a second measure. 

Now reboot your pc.

# Nope. Didnt work! So I deviced my own exploit aabusing root privileges to fix the issue. Down here. 

Make a ROOT and keep the character barely even used
Holy crap. It reanimated itself...

```
root@localhost:~# xset q | grep bell
  bell percent:  0    bell pitch:  400    bell duration:  100
```

Like literally. This thing wont die after I wiped the files. It wants to persist.

```
root@localhost:~# sh echo-kill-pcspkr.sh 
[Wed Feb 28 00:34:34 2018] input: PC Speaker as /devices/platform/pcspkr/input/input8
[Wed Feb 28 00:34:34 2018] snd_hda_intel 0000:00:1f.3: enabling device (0000 -> 0002)
[Wed Feb 28 00:34:35 2018] snd_hda_intel 0000:00:1f.3: bound 0000:00:02.0 (ops i915_audio_component_bind_ops [i915])
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0: autoconfig for ALC3246: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    hp_outs=1 (0x21/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    mono: mono_out=0x0
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    inputs:
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headset Mic=0x19
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headphone Mic=0x1a
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Internal Mic=0x12
[Wed Feb 28 00:34:34 2018] input: PC Speaker as /devices/platform/pcspkr/input/input8
[Wed Feb 28 00:34:34 2018] snd_hda_intel 0000:00:1f.3: enabling device (0000 -> 0002)
[Wed Feb 28 00:34:35 2018] snd_hda_intel 0000:00:1f.3: bound 0000:00:02.0 (ops i915_audio_component_bind_ops [i915])
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0: autoconfig for ALC3246: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    hp_outs=1 (0x21/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    mono: mono_out=0x0
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    inputs:
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headset Mic=0x19
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headphone Mic=0x1a
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Internal Mic=0x12
[Wed Feb 28 00:34:34 2018] input: PC Speaker as /devices/platform/pcspkr/input/input8
[Wed Feb 28 00:34:34 2018] snd_hda_intel 0000:00:1f.3: enabling device (0000 -> 0002)
[Wed Feb 28 00:34:35 2018] snd_hda_intel 0000:00:1f.3: bound 0000:00:02.0 (ops i915_audio_component_bind_ops [i915])
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0: autoconfig for ALC3246: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    hp_outs=1 (0x21/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    mono: mono_out=0x0
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    inputs:
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headset Mic=0x19
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headphone Mic=0x1a
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Internal Mic=0x12
sh: echo: I/O error
[Wed Feb 28 00:34:34 2018] input: PC Speaker as /devices/platform/pcspkr/input/input8
[Wed Feb 28 00:34:34 2018] snd_hda_intel 0000:00:1f.3: enabling device (0000 -> 0002)
[Wed Feb 28 00:34:35 2018] snd_hda_intel 0000:00:1f.3: bound 0000:00:02.0 (ops i915_audio_component_bind_ops [i915])
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0: autoconfig for ALC3246: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    hp_outs=1 (0x21/0x0/0x0/0x0/0x0)
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    mono: mono_out=0x0
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:    inputs:
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headset Mic=0x19
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Headphone Mic=0x1a
[Wed Feb 28 00:34:35 2018] snd_hda_codec_realtek hdaudioC0D0:      Internal Mic=0x12


```

It even reset it's FUCKING VOLUME.


# FINAL WORKING TRY, DOWNGRADE THE OWNER OF THE BELLLets try another method. Lets try to change ownership of these cursed files to a underprivileged user.

Make a user.

```
adduser losersocalleddeveloper

```

And DO NOT GIVE HIM ANY ADDITIONAL PERMISSIONS.

1. No sudo!
2. No root!
3. No su

Now, lets give this disgraced Linux developer his shitty bell back. Make another script, called your-fired.sh

```
#!/bin/sh
SHELL=/bin/bash

for shitcode in $(cat shitlist)
        do chmod 400 $shitcode
        chown loserlinuxdeveloper $shitcode
        done
sudo blacklist "blacklist pcspkr" > /etc/modprobe.d/pcspkr-blacklist.conf
sudo depmod -a
sudo usermod -u 9999 loserlinuxdeveloper
```

and have it cronjobbed at reboot. With another blacklist one-liner.

```
chmod 777 your-fired.sh
cp -r your-fired.sh /usr/local/bin
crontab -e
```

in crontab add

```
@reboot /bin/sh /usr/local/bin/your-fired.sh
* * * * * /bin/sh /usr/local/bin/your-fired.sh
```

then 

```sudo update-initramfs -u```

```service crontab restart```

This sets it to both not let the "dev" execute his crappy bell, BOTH on...

1. Each reboot
2. Each 24 hours at noon

Just in case... Linux gets sneaky on ya. And tries to make the bell run as root or something as what the /dev/input file implies.

If you have to, you may have to mess with /etc/passwd if the speakers beep again
```
nano /etc/passwd
CTRL+[\]
CTRL+[W]
enter 'losersocalleddeveloper'
Observe the permissions. Change the permissions from x:1000:1000 to a privilege level even lower. Tip is HIGHER THE NUMBER, THE LOWER THE PRIVILEGE. 

A root system user has a gid:0 and pid:0.
A new user without sudo privileges has gid:1000 and pid:1000
You can set it to over 9000 I believe. 

Most services are between 2000 to 9000 in in both gid and pid so they only are able to manage their own services.
```

But this can break many new recent versions of Linux (that is not Raspian). It does work in both upgrading and downgrading users in Raspberry Pi's default OS. But not sure about Debian.

Try usermod first

```usermod -u loserlinuxdeveloper```

YEEEAH! OWNED BIATCH. CAN'T EXECUTE YOUR DUMB BELL CAN YA!!!!

```
-r-------- 1 loserlinuxdeveloper root 1 Feb 28 01:38 /lib/modules/4.9.0-4-amd64/kernel/drivers/input/misc/pcspkr.ko
-r-------- 1 loserlinuxdeveloper root 1 Feb 28 01:38 /lib/modules/4.9.0-6-amd64/kernel/drivers/input/misc/pcspkr.ko
lrwxrwxrwx 1 root root 9 Feb 28 00:34 /dev/input/by-path/platform-pcspkr-event-spkr -> ../event7
-r-------- 1 loserlinuxdeveloper root 1 Feb 28 01:38 /lib/modules/4.9.0-5-amd64/kernel/drivers/input/misc/pcspkr.ko
```


Then reboot. Hope nothing breaks. 


I mean Debian is a great OS with great stability, but I am considering ditching it in favor of Ubuntu if this beep shit persists.


The Working Step.

1. add anew user just to have it control the account. This will be the only thing ever.
2. NO MORE PRIVILEGE UPGRADES We  actually are going to downgrade the fake user's erpmissoions, sok that either it will not be able to sudo to touch the bell or even participate in executing it, even though iti is his
4. Now modify user uid to 9999 to make it impossible for him to sudo
5. Chmod the files to 0400
6. Restart


