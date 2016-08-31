Sources: 

- https://www.sidorenko.io/blog/2014/11/04/yubikey-slash-openpgp-smartcards-for-newbies/
- https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/
- https://emanuelduss.ch/2015/01/sicheres-gnupg-setup-primary-key-offline-speichern/

Master Key erstellen (wird verwendet um Subkeys zu signieren)
1) ```gpg2 --gen-key```
 a) ```4``` RSA (sign only)
 b) key ```4096```
 c) 3 Jahre: ```3y``` und mit "Ja" bestätigen
 d) ```Vorname Nachname``` eingeben
 e) ```em@i.l``` Adresse eingeben
 f) Kommentar sollte leer gelassen werden
 g) Fertig
 h) Passphrase eingeben (Beliebige Zahl)

Subkeys erstellen
Maximale subkey-grösse: Yubikey Neo: 2048, Yubikey 4: 4096

3) ```gpg2 --expert --edit-key [e-mail adresse oder key id]```
  Subkey zum signieren
  a) ```addkey```
  b) ```4``` (RSA sign only / nur signiren/beglaubigen)
  c) ```2048``` oder ```4096```
  d) ```3y```
  e) Ja (wirklich erzeugen)

  Subkey zum verschlüsseln
  a) ```addkey```
  b) ```6``` (encrypt only / RSA nur verschlüsseln)
  c) ```2048``` or ```4096```
  d) ```3y```
  e) Ja (wirklich erzeugen)

  Subkey zum authentifizieren
  a) ```addkey```
  b) ```8``` (RSA Set your own capabilities / Leistungsfähigkeit selber einstellen)
  c) Nur Authentifizieren auswählen: Im Text oberhalb wird angezeigt, was aktuell angewählt ist.
     Solange togglen bis nur noch Authentification selektiert ist.
  d) Q (quit)
  c) ```2048``` or ```4096```
  d) ```3y```
  e) Ja (wirklich erzeugen)

  Neue Subkeys speichern
  a) ```save```


Weitere UID hinzufügen (E-Mail Adressen)
2) ```gpg2 --expert --edit-key [e-mail adresse oder key id]```
  a) adduid
  b) ```Vorname Nachname``` eingeben
  c) ```em@i.l``` Adresse eingeben
  d) Kommentar kann leer gelassen werden
  g) Fertig

# TODO: Generate revoke keys

Trust Level für neue UID ändern
3) gpg2 --edit-key [e-mail adresse oder key id]
  a) uid <UID nummer> eingeben. Dies lässt dich eine spezielle UID ändern
  b) trust
  c) 5
  d) save

Private Keys auf verschlüsseltem USB Stick speichern
4) gpg2 --export-secret-key --output [filename]
4) gpg2 --export-secret-subkey --output [filename]

Yubikey Modus setzen
5) https://www.yubico.com/2012/12/yubikey-neo-openpgp/
  a) Yubikey Personalization Tool installieren (CLI)
  b) Terminal öffenen und ykinfo -v eingeben. Wird der yubikey angezeigt kann er konfiguriert werden.
  c) Mode setzen: ykpersonalize -m82
  d) yes

Yubikey PIN setzen
6) gpg2 --card-edit
  a) In den Admin Modus wechseln: admin
  b) passwd
  c) Neuer PIN und Admin PIN setzen
  d) Bevor der PIN geändert werden kann muss der Default PIN eingegeben werden
     - 123456 => Default des normaler PIN
     - 12345678 => Default des admin PIN


Subkeys auf Yubikey übertragen
6) gpg2 --card-status
6) gpg2 --expert --edit-key [e-mail adresse oder key id] (Achtung muss untoggled gemacht werden)
    a) toggle
    b) Erster Subkey anwählen: key <Key Nummer>
    c) keytocard
    d) Korrekten Key zuweisen: Sign nach in den Sign Slot, usw.
    e) Passcode eingeben

Private Schlüssel auf Notebook löschen
7) gpg2 --delete-secret-keys [master key id]

Schlüssel auf Keyserver hochladen
8) gpg2 --send-keys [master key id]


scdaeomon einrichten:
https://blog.josefsson.org/2015/01/02/openpgp-smartcards-and-gnome/
