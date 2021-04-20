# Installing Fonos

Note: the rest of this tutorial assumes you will set the hostname of
your Pi to be `fonos`. Feel free to change it to whatever you want.

If you're setting up more than one Pi, be sure to use a different hostname for each one.


## Raspberry Pi base install

[Download] the Raspberry Pi OS Lite image and burn it to your SD card
with Etcher, `dd` or your own favorite method.

[Download]: https://www.raspberrypi.org/software/operating-systems/
With Etcher CLI, for example:

```bash
sudo etcher 2021-03-04-raspios-buster-armhf-lite.img --drive /dev/mmcblk0
```

Then, while the card is still in the drive, mount the first partition:

```bash
sudo mkdir -p /mnt/pi
sudo mount /dev/mmcblk0p1 /mnt/pi
```

Enable SSH by creating this file:

```bash
sudo touch /mnt/pi/ssh
```

If the Pi will be connected over WiFi, create a `wpa_supplicant.conf` file:

```bash
sudo tee /mnt/pi/wpa_supplicant.conf <<EOF
network={
	ssid="My Clever WiFi Name"
	psk="my super dope secret wifi password!!!"
}
EOF
```

That file will be moved (to `/etc/wpa_supplicant/`) when the Pi boots.

If the Pi will be connected over Ethernet, you don't need to create
that file or do anything special, it will work automatically.

Then, unmount the first partition:

```bash
sudo umount /mnt/pi
```

Proceed and mount the second partition, so that we can set the hostname:

```bash
sudo mount /dev/mmcblk0p2 /mnt/pi
```

Replace the default hostname with our custom one:

```bash
sudo sed -i s/raspberrypi/fonos/ /mnt/pi/etc/hosts /mnt/pi/etc/hostname
```

Finally, unmount the partition:

```bash
sudo umount /mnt/pi
```

You can then remove the SD card, insert it into the Raspberry Pi,
and power the Raspberry Pi.


## Logging into the Raspberry Pi

After a minute, the Raspberry Pi should be up and running.

You can check that it's correctly connected to the network by running:

```bash
ping fonos.local
```

Note: If the Pi isn't responding after a few minutes, try connecting to
your router's web interface to see if the device appears there. If it
doesn't, it might help to unplug the Pi and plug it back in.

Then connect with SSH:

```bash
ssh pi@fonos.local
```

The default password is `raspberry`.


## SSH key authentication

_If you don't mind typing a password every time you SSH to the Pi, you can skip this step._

From the Pi, create the `~/.ssh/authorized_keys` file and add your public SSH key. If you don't know how to do this, see the first few steps at the [GitHub tutorial on SSH keys](https://help.github.com/articles/connecting-to-github-with-ssh/).

Exit, then SSH to the Pi again to make sure you're not prompted for a password.

Once you've verified you can `ssh pi@fonos.local` without being prompted for a password, you can disable password login on the Pi:

`echo "PasswordAuthentication No" | sudo tee -a /etc/ssh/ssh_config`

SSH password authentication will be disabled on the next reboot.


## Deployment

From your **local machine**, complete the following steps:

### Install Ansible (version 2.0 or above)

See instructions for installing Ansible [here](http://docs.ansible.com/ansible/intro_installation.html).

### Clone this repo

```bash
git clone git@github.com:soulshake/fonos.git && cd fonos
```

### Create your Ansible inventory file (`hosts`)

From the root of the repo you just cloned, copy `hosts.sample` to `hosts` and modify it:

- add your own `spotify_username` and `spotify_password`, and credentials for other services you wish to enable
- replace the hosts under `[fonos]` with the hostname you chose earlier plus a `.local` extension (in our case, `fonos.local`)

The resulting `hosts` file should look something like this (if you have Pis with the hostnames `fonos` and `fonos2`):

<pre>
[fonos]
<b>fonos.local</b>
<b>fonos2.local</b>

[fonos:vars]
spotify_username=<b>your.spotify.username</b>
spotify_password=<b>yourSpotifyPa$$word</b>
</pre>

If you want to provision more Pis later, just add their hostnames under `[fonos]`.

### Run the Ansible playbook

```bash
ansible-playbook playbook.yml -i hosts
```

Once the playbook has completed, mopidy should be accessible at [http://fonos.local:6680/mopidy/](http://fonos.local:6680/mopidy/).

Note: For some reason, the playbook sometimes fails the first time at the "enable systemd units" step. If this happens, retry by running:

`ansible-playbook playbook.yml -i hosts --start-at-task="enable systemd units"`
