# Fonos

Fonos is a music jukebox that runs on a Raspberry Pi.

It can be controlled from a mobile app (any app using the MPD protocol,
for instance [MPD Remote]) or a web browser.

It is similar to [Pi MusicBox]. When we started working on Fonos,
Pi Musicbox didn't support the Raspberry Pi 3. Furthermore, we wanted
to implement a multi-room system, with multiple Raspberry Pis streaming
sound to each other.

Today, Pi Musicbox supports the Raspberry Pi 3, so you might want
to use it instead, as it seems to have more features. But Fonos
is here if you want to hack around! 😎

[Pi MusicBox]: https://www.pimusicbox.com/
[MPD Remote]: https://play.google.com/store/apps/details?id=net.prezz.mpr


## Quick start

1. Install [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) on a Raspberry Pi named `fonos`
2. Install your SSH key in the `pi` account
3. Make sure that you can log in (without password) with `ssh pi@fonos.local`
4. Clone this git repo on your local machine
5. Copy `hosts.sample` to `hosts` and customize that `hosts` file
6. Install Ansible on your local machine
7. Run `ansible-playbook playbook.yml -i hosts`
8. Connect to http://fonos.local:5555/

See [INSTALL.md] for detailed instructions.


## What's running, and how

The Ansible playbook will install and run the following components:
- Mopidy, the music jukebox
- PulseAudio, the sound server
- PaWebControl, a web interface to adjust volume with pulseaudio
- AuRevoir, a web browser for zeroconf/bonjour services

Each component starts through a systemd user unit.

Fonos installs some system packages, but it shouldn't touch any
system configuration file.

The Python components (Mopidy and AuRevoir) run from a virtualenv
located in `/home/pi/fonos/env`.

PaWebControl is a very basic PHP app. It runs directly with the PHP CLI
web server (without a "real" web server like Apache or NGINX).


## Troubleshooting

A few things you can do from the Pi...

Check Mopidy's configuration:
```bash
source /home/pi/fonos/env/bin/activate
mopidy config
```

Check the status of the various components:
```bash
systemctl --user status
```

Check, reload, or restart Mopidy in particular:
```bash
systemctl --user status mopidy
systemctl --user reload mopidy
systemctl --user restart mopidy
```

Check the status of PulseAudio (it looks like it can crash sometimes):
```bash
systemctl --user status pulseaudio
```

View logs of Mopidy or PulseAudio:
```bash
sudo journalctl _SYSTEMD_USER_UNIT=mopidy.service
sudo journalctl  _SYSTEMD_USER_UNIT=pulseaudio.service
```


### "I can view the web interfaces but nothing is playing"

Ensure your credentials are correct in the output of `mopidy config` as described above.

Try downloading an mp3 directly to the Pi:

```bash
wget https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3 /home/pi/fonos
```

Files in the `fonos` directory should show up under `Files` in the [Moped interface](http://fonos.local:6680/moped), for example. If it plays through your speakers, there might be an issue with your credentials for the service you're trying to play through (e.g. Spotify/Soundcloud/etc).


### "The interface shows that it's playing, but I don't hear any sound"

- Ensure your Pi is connected to your speaker via audio cable.
- Ensure your speaker is plugged in and on.

This may sound obvious, but it happens to the best of us :)


## Combining sinks


These are some experimental notes that may or may not work.

On the Pi, list sinks by name:

```
pacmd list-sinks | grep -i name:
	name: <alsa_output.0.analog-stereo>
	name: <tunnel.vorpal.local.alsa_output.pci-0000_00_03.0.hdmi-stereo>
	name: <tunnel.vorpal.local.alsa_output.pci-0000_00_1b.0.analog-stereo>
	name: <tunnel.vorpal.local.combined>
	name: <tunnel.fonos2.local.alsa_output.0.analog-stereo>
	name: <tunnel.fonos2.local.alsa_output.0.analog-stereo.2>
```

Create a combined output between our Pi's output (`alsa_output.0.analog-stereo`) and the corresponding output on the other pi (`tunnel.fonos2.local.alsa_output.0.analog-stereo`). (There's duplicates (with a `.2` suffix) because of ipv6.)

Create a new combined sink:

```
pacmd load-module module-combine-sink \
  sink_name=combined \
  slaves="alsa_output.0.analog-stereo,tunnel.fonos2.local.alsa_output.0.analog-stereo"
```

Now if you run `pacmd list-sinks | grep -i name:` again, you'll see the new sink:

```
	name: <combined>
```

A snippet to combine all USB sinks:

```
pacmd load-module module-combine-sink \
  sink_name=combined \
  slaves=$(pacmd list-sinks 
    | sed -n 's/^\s*name: <\(.*\)>$/\1/p' \
    | grep -e alsa_output.usb \
    | tr "\n" ",")
```

Open pavucontrol from your host with the `PULSE_SERVER` environment variable set to your Pi hostname:

`PULSE_SERVER=fonos.local pavucontrol`

In the **Playback** tab, you should be able to select the combined output.

All your speakers should now, in theory, be producing sound.
