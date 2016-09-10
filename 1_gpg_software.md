# Software
To manage and use your GPG-keys and your yubikey, some tools are required.

## Fedora

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

## Arch Linux

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
