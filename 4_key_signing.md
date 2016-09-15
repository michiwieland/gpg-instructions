# TODO: GPG key signing

The signing of foreign keys must be done with your master key. This should again be done in a secure environment (see chapter 1).

TODO: This is work in progress (missing: usb-stick pubkey transfer, tails startup, encrypted usbdisk unlocking)

1. Check that the fingerprint matches with the paper in your hand: ```gpg2 --fingerprint otherkey```
2. ```gpg2 --export otherkey | gpg2 --homedir /mnt/usbdisk/... --import```
3. ```gpg2 --homedir /mnt/usbdisk/... --sign-key otherkey```
4. ```gpg2 --homedir /mnt/usbdisk/... --send-keys otherkey```
5. ```gpg2 --homedir /mnt/usbdisk/... --export otherkey | gpg2 --import```
