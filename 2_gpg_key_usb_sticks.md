## USB-storage to save private key

In case you loose your Yubikey, or need to do operations with your master key (like signing a key), it is important to have backups of your private keys.

It is recommended to create two USB-sticks to store your key in two physical separate places.

### Fill USB-stick with random data

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
