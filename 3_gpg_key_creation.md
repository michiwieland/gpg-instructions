# GPG key creation

## Prerequisities

- 1 or (better) more USB drives that permanently hold your privat keys.
- 1 USB drive to transfer data from your safe to your insecure environment
- Yubikey 4 or Yubikey Neo
- Tails Live CD https://tails.boum.org/
- Computer with CD drive (and no plugged network cable)

### Secure Tails environment

To create your keys, it is a good idea to have a live linux like Tails. Download the Tails ISO-file from the [Tails website](https://tails.boum.org) and burn it to a CD-ROM; remove any network cables, boot the CD and voilá, you have a secure environment.

## Encrypted usb drive

In case you loose your Yubikey, or need to do operations with your master key (like signing a key), it is important to have backups of your private keys.

It is recommended to create two USB-sticks to store your key in two physical separate places.

### Fill USB-stick with random data

Overwrite USB disks with random data
```bash
cryptsetup open --type plain /dev/sdb container --key-file /dev/random
# check whether /dev/mapper/container exists
dd if=/dev/zero of=/dev/mapper/container

cryptsetup close container
```

### Create Encrypted partition

```bash
cryptsetup -v --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase luksFormat /dev/sdb

cryptsetup open --type luks /dev/sdb usbstick

mkfs.ext4 /dev/mapper/usbstick

mkdir /mnt/usbstick && mount -t ext4 /dev/mapper/usbstick /mnt/usbstick

# When you're done:
umount /mnt/usbstick
cryptsetup close usbstick
```

## Key creation

### Create master key

Your master key is used to sign, "issue" and revoke your subkeys. It makes handling of key creation / revokation much easyer and (quite) painless.

```gpg2 --gen-key```
1. Choose ```4``` RSA (sign only)
2. Key length ```4096``` (always 4096)
3. Key is 3 Years valid (can be extended later): ```3y``` and confirm with ```Yes```
4.  Insert key contact name: ```Firstname Surname```
5.  insert your primary email address: ```em@i.l``` 
6.  Leave comment field empty
7.  ```save``` (TODO:is this the command?)
8.  Type passphrase (>20 random characters, make shure to store them savely (best on paper))


### Create subkeys

Maximum key sizes (RSA):

- Yubikey Neo: 2048
- Yubikey 4: 4096

```gpg2 --expert --edit-key em@i.l``` (primary email address)
1. Create signing subkey
  1. ```addkey```
  2. Key type: ```4``` (RSA sign only)
  3. Key length: ```2048``` or ```4096``` (depending on your yubikey-series)
  4. Validity: ```3y```
  5. Confirm with ```yes```
2. Create encrypting subkey
  1. ```addkey```
  2. Key type: ```6``` (encrypt only)
  3. Key length: ```2048``` or ```4096```
  4. Validity: ```3y```
  5. Confirm with ```yes```
3. Create authentication subkey
  1. ```addkey```
  2. Key type: ```8``` (RSA Set your own capabilities)
  3. Choose authentication only (toggle all options accordingly). The current selection is shown above the choices.
  4. ```q``` (quit selection)
  5. Key length: ```2048``` or ```4096```
  6. Validity: ```3y```
  7. Confirm with ```yes```
4. Save new subkeys with ```save```.

### Add UIDs

Add more UIDs (email addresses). Your primary address inserted above is already present and should not be entered another time.

```gpg2 --expert --edit-key em@i.l``` (primary email address)
1. ```adduid```
2. Insert ```Firstname Surname```
3. Insert your additional email address ```em@i.l```
4. Leave comments field empty
5. ```save``` (TODO: really?)


Change trust level for the new UIDs:

```gpg2 --expert --edit-key em@i.l``` (primary email address)
1. ```uid <NUMBER>``` for each of your created subkeys (TODO: How to display selection etc?)
2. Set the ```trust``` level for your addresses
3. Give your adresses ultimate trust: ```5```
4. ```save```

### TODO: Generate revokation certificates

TODO.

### Save keys on usb drive

As recommended in the usb drive section, export the secret key on every of your drives.

Replace *YYYY-MM* with the current date.

```bash
gpg2 --export-secret-key --output [/mnt/usbstick/secret_masterkey.gpg]
gpg2 --export-secret-subkey --output [/mnt/usbstick/secret_subkeys_YYYY-MM.gpg]
sync
```

It is essential that the exports above worked, otherwise do NOT proceed with the following step but resolve the issue first!

### Remove private master key

Before transfering you subkeys to your yubikey, it is recommended to remove the master key. As there is no command to do that directly, take following steps:


1. Export secret subkeys only ```gpg2 --export-secret-subkeys --output /tmp/secret_subkeys.gpg```
2. Remove all secret keys from keyring ```gpg2 --delete-secret-key em@i.l``` (with your primary email)
3. Import the secret subkeys ```gpg2 --import /tmp/secret_subkeys.gpg```

## Yubikey

### Yubikey preparations

#### Set Yubikey mode
1. Open a terminal and type ```ykinfo -v```. If the Yubikey version is displayed, you can continue with the following steps.
2. Set the Yubikey mode: ```ykpersonalize -m82```
3. Confirm with ```yes``` (TODO: "yes" - really?)

#### Set Yubikey pin 
Yubikey PINs may (and should) contain all characters (and not just 0-9).  
Your Yubikey has following default pins:

- primary PIN: ```123456```
- admin PIN: ```12345678```

```gpg2 --card-edit```
1. Change to admin mode: ```admin```
2. ```passwd```
3. Set new PIN. This PIN is for your daily use; make shure it is >15 characters, and that you can memorize it.
4. Set new admin PIN. It is only used in special cases - make shure to make some random sequence of at least 20 characters.
5. Save the key sequences in the file /mnt/usbstick/yubikey-pins.txt

TODO: The last section should be documented in a more step-by-step style.

#### Transfer subkeys on Yubikey
1. Reinsert your Yubikey and make shure it is ready with ```gpg2 --card-status```
2. ```gpg2 --expert --edit-key em@i.l``` (primary email address)
3. ```toggle``` public/privat key mode
4. Choose first subkey: ```key <NUMBER>``` with the number from the list
5. ```keytocard``` (TODO: or is it ```addtocard```?)
6. Choose correct key slot (sign key to sign slot etc.).
7. Insert your gpg-key password
8. Remove key selection with ```key <NUMBER>```
9. Repeat steps 4 to 8 for every subkey (signing, encryption, authentication)

#### Export public key to unencrypted USB drive.

1. Delete the secret keys from your keyring: --delete-secret-keys em@i.l``` (primary email address)
2. Export Public keys ```gpg2 --armor --export em@i.l > /mnt/insecure_usb/publickey.pem``` (primary email address)



## Setup on your usual system

### Set systemd environment variables

In the following sections, we assume you are running a system with the `systemd` init daemon (like virtually all modern linux distributions do).

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

### TODO: GPG Configuration

- recommended configuration of PGP
- https://sks-keyservers.net/sks-keyservers.netCA.pem

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
# TODO: What does this scdaemon configuration do exactly again?
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


### Import & Upload your public key to the keyserver

1. Mount the USB drive with your public key to ```/mnt/insecure_usb/```
2. Import your public key: ```gpg2 --import /mnt/insecure_usb/publickey.pem```
3. Trust your public key: ```gpg2 --expert --edit-key em@i.l``` (primary email address)
4. ```trust```
5. ``5``` (ultimate trust yourself?)
6. ```save```
7. Upload your key to the keyserver: ```gpg2 --send-keys em@i.l``` (primary email address)


## Sources

- yubikey: https://www.sidorenko.io/blog/2014/11/04/yubikey-slash-openpgp-smartcards-for-newbies/
- yubikey: https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/
- yubikey: https://www.yubico.com/2012/12/yubikey-neo-openpgp/
- gpg-setup: https://emanuelduss.ch/2015/01/sicheres-gnupg-setup-primary-key-offline-speichern/
- scdaemon: https://blog.josefsson.org/2015/01/02/openpgp-smartcards-and-gnome/
- TODO?: https://blog.habets.se/2013/02/GPG-and-SSH-with-Yubikey-NEO
- TODO?: https://www.whonix.org/wiki/Air_Gapped_OpenPGP_Key
