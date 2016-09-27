# Sources

- yubikey: https://www.sidorenko.io/blog/2014/11/04/yubikey-slash-openpgp-smartcards-for-newbies/
- yubikey: https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/
- yubikey: https://www.yubico.com/2012/12/yubikey-neo-openpgp/
- gpg-setup: https://emanuelduss.ch/2015/01/sicheres-gnupg-setup-primary-key-offline-speichern/
- scdaemon: https://blog.josefsson.org/2015/01/02/openpgp-smartcards-and-gnome/
- TODO?: https://blog.habets.se/2013/02/GPG-and-SSH-with-Yubikey-NEO
- TODO?: https://www.whonix.org/wiki/Air_Gapped_OpenPGP_Key
- TODO?: https://blog.habets.se/2016/01/Yubikey-4-for-SSH-with-physical-presence-proof


# Computer Setup

To create secure GPG-Keys, we are going to use two system environments:

- Your primary operating system
- A secure, non permanent linux-live system called Tails Linux.

At first, we make your primary operating system ready to use GPG and your Yubikey.

Before starting with this guide, be shure to have following physical things present:

- 1 or (better) multiple USB drives that permanently hold your privat keys.
- Another USB drive to transfer data from your safe to your insecure environment
- Yubikey 4 or Yubikey Neo
- An empty CD-ROM recommended or another USB drive (to burn a Tails Linux live system)
- Computer with CD drive (or *at least* 2 USB ports)

## Software
To manage and use your GPG-keys and your yubikey, some tools are required:

### Fedora Linux

```bash
# GPG
sudo dnf install gnupg2 gnupg2-smime

# Smartcard-Tools
sudo dnf install pcsc-tools opensc ccid

# Yubikey-Tools
sudo dnf install yubikey-personalization-gui ykpers ykclient

# GPG keyring updates over the onion
sudo dnf install parcimonie.sh tor tor-arm

# Pass password manager
sudo dnf install pass
```

### Arch Linux

```bash
# GPG
sudo pacman -S gnupg2

# Smartcard-Tools
sudo pacman -S pcsc-tools opensc ccid

# Yubikey-Tools
yaourt -S yubikey-personalization-gui
sudo pacman -S yubico-c-client

# GPG keyring updates over the onion
sudo pacman -S tor arm
yaourt -S parcimonie-sh-git

# Pass password manager
sudo pacman -S pass
```

## GPG-Configuration

### Set systemd environment variables

In the following sections, we assume you are running a linux system with the `systemd` init daemon (like virtually all modern linux distributions do).

Systemd has the possiblity to start daemons on the login or logout process of your user. To use the commands below, clean environment settings are required:

```bash
mkdir -p ~/.config/systemd/user/

cat <<<"__EOF__" > ~/.config/systemd/user/
[Unit]
Description=User environment
Before=default.target

[Service]
Type=oneshot
ExecStart=-/usr/bin/systemctl --user set-environment \
        GNUPGHOME=%h/.gnupg \
        SSH_AUTH_SOCK=/run/user/`id --user`/gnupg/S.gpg-agent.ssh \
        GPG_AGENT_INFO=/run/user/`id --user`/gnupg/S.gpg-agent.ssh

[Install]
WantedBy=default.target
__EOF__

systemctl --user enable env.service
```

TODO: This configuration is currently only testet on Fedora Linux...

### GPG Configuration


```bash
wget -O ~/.gnupg/sks-keyservers.netCA.pem https://sks-keyservers.net/sks-keyservers.netCA.pem 

cat <<'__EOF__' > ~/.gnupg/dirmngr.conf
##################################################################
## Keyserver
##################################################################
hkp-cacert ~/.gnupg/sks-keyservers.netCA.pem
__EOF__

cat <<'__EOF__' > ~/.gnupg/gpg.conf
##################################################################
## General configuration
##################################################################
## Use a agent to save user key
use-agent

##################################################################
## Key creation and signatures
##################################################################
## Don't disclose the version
no-emit-version

## Don't add additional comments (may leak language, etc)
no-comments

## include an unambiguous indicator of which key made a signature:
## (see http://thread.gmane.org/gmane.mail.notmuch.general/3721/focus=7234)
sig-notation issuer-fpr@notations.openpgp.fifthhorseman.net=%g

##################################################################
## Keyserver
##################################################################
## Proxy settings (use sks-keyservers tor if available)
keyserver hkps://hkps.pool.sks-keyservers.net hkp://qdigse2yzvuglcix.onion
keyserver-options auto-key-retrieve

## When creating a key, individuals may designate a specific keyserver to use to pull their keys from.
## The above option will disregard this designation and use the pool, which is useful because (1) it
## prevents someone from designating an insecure method for pulling their key and (2) if the server
## designated uses hkps, the refresh will fail because the ca-cert will not match, so the keys will
## never be refreshed.
keyserver-options no-honor-keyserver-url

##################################################################
## Crypto
##################################################################
## list of personal digest preferences. When multiple digests are supported by
## all recipients, choose the strongest one
personal-cipher-preferences AES256 AES192 AES CAST5
#TODO: We should already add Curve 25519, even if not implemented yet

## Use SHA512 as digest
cert-digest-algo SHA512

## Our preferences if SHA512 is not possible
personal-digest-preferences SHA512 SHA384 SHA256 SHA224

## List our Crypto preferences
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed

##################################################################
## Display options
##################################################################
## We want to force UTF-8 everywhere
display-charset utf-8

## when outputting certificates, view user IDs distinctly from keys:
fixed-list-mode

## when listing certificates, also list fingerprint.
with-fingerprint

## long keyids are more collision-resistant than short keyids (it's trivial to make a key with any desired short keyid)
keyid-format 0xlong

## You should always know at a glance which User IDs gpg thinks are legitimately bound to the keys in your keyring:
verify-options show-uid-validity
list-options show-uid-validity

##################################################################
## Optional user settings
##################################################################
## Up to you whether you in comment it (remove the single # in front of
## it) or not. Disabled by default, because it causes too much complaints and
## confusion.

## Don't include keyids that may disclose the sender or any other non-obvious keyids
## This may slow down decryption, as all private keys must be tried.
#throw-keyids

#################################################################
__EOF__
```

### Key updates with parcimonie.sh

```parcimonie.sh``` is a script that periodically updates the GPG-public keys that are stored locally in a secure (and privacy aware) manner.

```bash
cat <<<'__EOF__' > .config/systemd/user/parcimonie.service
[Unit]
Description=GnuPG parcimonie key refresher

[Service]
ExecStart=/usr/bin/parcimonie.sh

[Install]
WantedBy=default.target
__EOF__

systemctl --user start parcimonie.service
systemctl --user enable parcimonie.service
```

### GPG agent and scdaemon

The GPG agent stores your Yubikey-PIN for some time, so that you don't have to type it every time you use it. In our setup, we also use this daemon for SSH-Authorization.

The scdaemon handles the communication between the yubikey (SmartCard) and the gpg-daemon.

```bash
cat <<<'__EOF__' > ~/.gnupg/gpg-agent.conf 
enable-ssh-support
default-cache-ttl 900
default-cache-ttl-ssh 900
allow-loopback-pinentry
__EOF__

# Create & enable daemon
cat <<<'__EOF__' > ~/.config/systemd/user/gpg-agent.service
[Unit]
Description=GnuPG private key agent
IgnoreOnIsolate=true

[Service]
Type=forking
ExecStart=/usr/bin/gpg-agent --daemon --enable-ssh-support
Restart=on-abort

[Install]
WantedBy=default.target
__EOF__

systemctl --user start gpg-agent.service
systemctl --user enable gpg-agent.service
```

```bash
# TODO: What does this scdaemon ccid configuration do exactly again?
cat <<<'__EOF__' > ~/.gnupg/scdaemon.conf
disable-ccid
__EOF__

# TODO: How is the extecution of this eventhandler configured?
cat <<<'__EOF__' > ~/.gnupg/scd-event
#!/bin/sh
state=$8

# Make shure that the scdaemon is exitet when the card is removed.
# scdaemon has the bad habit to crash whenever you remove a card.
if [ "$state" = "NOCARD" ]; then
  pkill scdaemon
fi
__EOF__

```

## Secure Tails environment

To create your keys, you should use a isolated system that is not bound to any maleware that may be on your computer already. Tails is a linux live system that is specifically designed for such tasks.

Download the Tails ISO-file from the [Tails website](https://tails.boum.org) and burn it to a CD-ROM. After that, boot the system from the disk. (you could also write the ISO-image to a USB drive if you don't have a CD drive).

Once started, define your root-password, login and open a system shell. Get root with `sudo -s` and your just choosen password.


### Install software

TODO: Paket names are from Fedora, must be adopted to debian. (possibly additional repo for yubikey required)

```bash
# GPG
apt-get install gnupg2 gnupg2-smime

# Smartcard-Tools
apt-get install pcsc-tools opensc ccid

# Yubikey-Tools
apt-get install yubikey-personalization-gui ykpers ykclient
```

After you installed this software from the internet, please remove the network connection (if possible by physical plugging).


## Encrypted usb drive

In case you loose your Yubikey, or need to do operations with your master key (like signing another key), it is important to have backups of your private keys.

It is recommended to create two USB disks to store your key in two physical separate places.

To use the following commands, you must replace ```/dev/DRIVE``` with your USB disk's identifier (like ```/dev/sdb```). To find out, which letter your USB disk has, type the command ```lsblk``` and find the appropriate disk size.

### Fill USB disks with random data

Overwrite USB disks with random data
```bash
cryptsetup open --type plain /dev/DRIVE container --key-file /dev/random
# check whether /dev/mapper/container exists
dd if=/dev/zero of=/dev/mapper/container

cryptsetup close container
```

### Create encrypted partition

```bash
cryptsetup -v --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase luksFormat /dev/DRIVE

cryptsetup open --type luks /dev/DRIVE usbdisk

mkfs.ext4 /dev/mapper/usbdisk

mkdir /mnt/usbdisk && mount -t ext4 /dev/mapper/usbdisk /mnt/usbdisk

# When you're done:
umount /mnt/usbdisk
cryptsetup close usbdisk
```

