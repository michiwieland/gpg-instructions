1) gpg2 --fingerprint BEBF7D2C

2) gpg2 --export BEBF7D2C | gpg2 --homedir /media/michi/Backup/ --import

3) gpg2 --homedir /media/michi/Backup/ --sign-key BEBF7D2C

4) gpg --homedir /media/michi/Backup/ --send-keys BEBF7D2C

5) gpg --homedir /media/michi/Backup/ --export BEBF7D2C | gpg --import
