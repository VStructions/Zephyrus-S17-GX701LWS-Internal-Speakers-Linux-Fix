# Zephyrus-S17-GX701LWS-Internal-Speakers-Linux-Fix
User-space patch for enabling the chronically troubled, Zephyrus S17's internal speakers, on Linux based OSs. The issue lies in sound card handling. For Windows, this laptop is given a proprietary version of the Realtek ALC296 driver. This patch is known to work in Ubuntu 26.04 LTS, but is not specific to it. Instructions below target Debian distributions using Pipewire.

Specifically, I believe there are issues with sound card:
- Initialization
- Waking from power-save mode
- Definition of active channel for volume control

## How to apply the patch
Install these Linux sound tools:
```
sudo apt install alsa-tools pipewire pipewire-pulse wireplumber
```

Clone this repo to a local directory, then enter that directory\
e.g. `cd /....../Zephyrus-S17-GX701LWS-Internal-Speakers-Linux-Fix`


Copy [alc294_gx701lws.fw](alc294_gx701lws.fw) to `/lib/firmware`:
```
sudo cp alc294_gx701lws.fw /lib/firmware/alc294_gx701lws.fw
```

Provide the fw file to the kernel, to configure the sound card during boot. Enter this file, :
```
sudo nano /etc/modprobe.d/alsa-base.conf
```
then append this line at the bottom:
```
options snd-hda-intel patch=alc294_gx701lws.fw
```

Block Linux from putting the card to powersaving mode (wake up glitches). Copy [disable_powersave.conf](disable_powersave.conf) to `/etc/modprobe.d`:
```
sudo cp disable_powersave.conf /etc/modprobe.d/disable_powersave.conf
```

Adjusting volume doesn't work and volume is most likely stuck at 100%. (I think, the card only exposes PCM channel volume control, while Ubuntu by default controls MASTER). Create this configuration directory for wireplumber:
```
mkdir -p ~/.config/wireplumber/wireplumber.conf.d/
```

Copy [51-volume-fix-alc294-gx701lws.conf](51-volume-fix-alc294-gx701lws.conf) to `~/.config/wireplumber/wireplumber.conf.d`:
```
cp 51-volume-fix-alc294-gx701lws.conf ~/.config/wireplumber/wireplumber.conf.d/51-volume-fix-alc294-gx701lws.conf
```

Enter this ALSA configuration file, :
```
sudo nano /usr/share/alsa-card-profile/mixer/paths/analog-output.conf.common
```
then (change the) or (create an) Element Master section, to this:
```
[Element Master]
switch = mute
volume = ignore
```

Lastly, because of how volume control is now set up and because of how headphones volume is controlled (when 3.5mm is plugged in Ubuntu controls PCM, the opposite from internal speakers!), headphones volume might be limited, depending on where the Master slider was previously left. To fix this, we set Master and Headphones volume to 100% and apply this configuration on every boot:
```
amixer set Master 100% && amixer set Headphone 100%
sudo alsactl store
```

### Reboot the laptop
Internal speakers, headphones audio and volume control should now work.
