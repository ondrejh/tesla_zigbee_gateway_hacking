Budem potřebovat 3V3 seriák a ethernet.

1) vstup do bootloaderu

Anobrž šmejdi maji upravenej bootloader, tak se nedá vstoupit do bootloaderu.
Nuže fígl je, že po startu, až to napíše první tabulku a začne zavádět jádro,
tak šlusneme nožky 5 a 6 na seriové flašce, tim to zblbneme zavádení jádra a
hopne to do bootloaderu.

2) flešnutí nového bootloaderu
   
Na PCčku si připravíme ethernetovou síťovku s adresou 192.168.1.97,
spustíme tftp server (pro uživatele profesionálního operačního systému např.
https://www.tftpd64.com), kterej bude umět dodat soubor boot.bin 

Pozn.: Pro ty co profesionální operační systém nemají nebo nechtějí:
Instalace a start tftp serveru
```
sudo apt install tftpd-hpa
sudo cp boot.bin /srv/tftp/
sudo chmod 644 /srv/tftp/boot.bin
sudo systemctl restart tftpd-hpa
```
Vypnuti serveru (aby tam pak nestrašil)
```
sudo systemctl stop tftpd-hpa
```

V bootloaderu zadáme příkaz (když tftp server běží a nabízí boot.bin):
tftp 0x80500000 boot.bin

A když všechno projde, tak se sosne bootloader a rovnou se flashne.
A tím máme krásnej, novej, voňavej bootloader, do kterého se dá už normálně
vskočit pomocí držení ESC při bootu.

3) nahrajeme jadérko

Na PCčku si připravíme tftp klienta, kterej se bude připojovat k 192.168.1.6 
(to je default adresa zadrátovaná v bootloaderu). Jako normělní lidi vlezeme
do bootloaderu pomocí ESC a TFTP klientem tam napereme soubor kernel.img. 
Uděje se kouzlo a reboot a začne to startovat.

Pozn.: Tady je zajímavé, že původní bootloader měl tftp klienta, zatímco nový, voňavý, dělá server. Nu což, změna je život.
Neprofesionální klient takto:
```
tftp 192.168.1.6
tftp> mode octet
tftp> put kernel.img
tftp> quit
```
Nebo takto:
```
tftp 192.168.1.6 -m octet -c put kernel.img
```
Ten přepínač octet je tam proto, že neprofesionální klient je až příliš chytrý a vyjednává se serverem velikosti bloků a podobné záležitosti. Jenže server je v tomto případě hloupý a tak tomu nerozumí. Skončí to SUCCESS! a v zápětí checksum error...
Jo a ještě jedna věc. I neprofesionální operační systémy dnes mívají firewall. A tftp je protokol, který nemá už 30 let nikdo rád.

4) rootfs

Velmi podobný postup, jen tam budem hulit soubor rootfs.bin

```
tftp 192.168.1.6 -m octet -c put rootfs.bin
```

5) userdata

Zase podobně, jen vhodny soubor userdata.bin

```
tftp 192.168.1.6 -m octet -c put userdata.bin
```
