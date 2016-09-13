# GPG key creation

## Create master key

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


## Create subkeys

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

## Add UIDs

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

## TODO: Generate revokation certificates

TODO.

## Save keys on usb drive

As recommended in the usb drive section, export the secret key on every of your drives.

Replace *YYYY-MM* with the current date.

```bash
gpg2 --export-secret-key --output [/mnt/usbstick/secret_masterkey.gpg]
gpg2 --export-secret-subkey --output [/mnt/usbstick/secret_subkeys_YYYY-MM.gpg]
sync
```

It is essential that the exports above worked, otherwise do NOT proceed with the following step but resolve the issue first!

## Remove private master key

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

1. Delete the secret keys from your keyring: ```gpg2 --delete-secret-keys em@i.l``` (primary email address)
2. Export Public keys ```gpg2 --armor --export em@i.l > /mnt/insecure_usb/publickey.pem``` (primary email address)



## Import & Upload your public key to the keyserver

1. Mount the USB drive with your public key to ```/mnt/insecure_usb/```
2. Import your public key: ```gpg2 --import /mnt/insecure_usb/publickey.pem```
3. Trust your public key: ```gpg2 --expert --edit-key em@i.l``` (primary email address)
4. ```trust```
5. ``5``` (ultimate trust yourself?)
6. ```save```
7. Upload your key to the keyserver: ```gpg2 --send-keys em@i.l``` (primary email address)

