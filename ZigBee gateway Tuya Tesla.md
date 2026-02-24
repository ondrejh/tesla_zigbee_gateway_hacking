![IMG20260221121425.jpg](img/IMG20260221121425.jpg)  
![IMG20260221133121.jpg](img/IMG20260221133121.jpg)

Nejdřív, když jsem zjistil že plošňák vypadá stejně, jsem doufal, že to půjde jako s [Lidl gateway SilverCrest - Lidl (Tuya) SmartHome Gateway Ohýbání]("Lidl gateway SilverCrest - Lidl (Tuya) SmartHome Gateway Ohýbání.md")). Jenže tady nejde přerušit bootloader, protože je to zamčené. Takže asi přichází v úvahu jen SPI Flash Dump ... NOR Flash GigaDevice 25Q127CSIG.

Nový firmware a postup flashování je v [repozitáři jnilo1/hacking-silvercrest-gateway](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway). SSH přístup nemám (to bych toto nedělal), a bootloader ESC stopnout nemůžu, jako v případě original Lidl gatewaye, takže zbývá [Method 3 - SPI Programmer (CH341A or Equvivalen)](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/tree/main/3-Main-SoC-Realtek-RTL8196E/30-Backup-Restore#-method-3--spi-programmer-ch341a-or-equivalent). Programátor ovšem taky nemám. Pokusím se ho nahradit RP2040 zero dle [https://github.com/stacksmashing/pico-serprog](https://github.com/stacksmashing/pico-serprog)... uf, nějak se mi to komplikuje...

- [x] sehnat hw
	- breadboard
	- rp2040 zero

- [x] flashnout programátor

```bash
git clone https://github.com/stacksmashing/pico-serprog.git
cd pico-serprog
mkdir build
cd build
cmake -DPICO_SDK_PATH=<path to pico sdk> ..
make
cp pico-serprog.u2f /media/.../RPI-RP2/
```

- [x] zapojit HW
![19ac0dbe96ea2f6723a3c43040565081.png](img/19ac0dbe96ea2f6723a3c43040565081.png)

| GPIO | Pico Pin | Function |
|------|----------|----------|
| 1    |    2     | CS       |
| 2    |    4     | SCK      |
| 3    |    5     | MOSI     |
| 4    |    6     | MISO     |

![15a8a9469aafeb0c7806f0ebebfc665b.png](img/15a8a9469aafeb0c7806f0ebebfc665b.png)

![IMG20260221171942.jpg](img/IMG20260221171942.jpg)

### Pokus o spojení:

```bash
$ flashrom -p serprog:dev=/dev/ttyACM0:115200,spispeed=12M -c GD25Q127C/GD25Q128C
flashrom v1.2 on Linux 5.15.0-170-generic (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
serprog: Programmer name is "pico-serprog"
Found GigaDevice flash chip "GD25Q127C/GD25Q128C" (16384 kB, SPI) on serprog.
No operations were specified.
```

Jo! Tedy na rozdíl od návodu je třeba secifikovat jiný programátor, ale také jiný čip, protože moje verze flashrom holý GD25Q128C nezná ... Ale při spuštění bez specifikace čipu (bez -c ...), se připojí, zjistí, a řekne že musím vybrat mezi dvěma. A jeden z nich je GD25Q127C/GD25Q128C... což odpovídá mému GD25Q127C, takže zatím cajk.

### Záloha flashky:

```bash
$ flashrom -p serprog:dev=/dev/ttyACM0:115200,spispeed=12M -c GD25Q127C/GD25Q128C -r tesla_gateway_backup.bin
flashrom v1.2 on Linux 5.15.0-170-generic (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
serprog: Programmer name is "pico-serprog"
Found GigaDevice flash chip "GD25Q127C/GD25Q128C" (16384 kB, SPI) on serprog.
Reading flash... done.
```

```bash
$ ls -l tesla_gateway_backup.bin
-rw-rw-r-- 1 ... 16777216 ... tesla_gateway_backup.bin
```

Jestli je to dobře? Kdo ví?

### Flashování

No ale co dál? Teď mám připojenou funkční 16MB flešku, ale všechny návody předpokládají, že mám ssh přístup. To nemám! To bych přece nevyndával tu flešku...

Možná že je nápověda tady:

Partition Layout
After migration:

```
0x000000-0x020000  mtd0  boot+cfg     (128 KB)   - Bootloader (unchanged)
0x020000-0x200000  mtd1  linux        (1.9 MB)   - Linux kernel
0x200000-0x420000  mtd2  rootfs       (2.1 MB)   - Root filesystem
0x420000-0x1000000 mtd3  jffs2-fs     (11.9 MB)  - User partition
```

No a ty soubory boudou asi: [boot.bin](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/blob/main/3-Main-SoC-Realtek-RTL8196E/31-Bootloader/boot.bin), [kernel.img](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/blob/main/3-Main-SoC-Realtek-RTL8196E/32-Kernel/kernel.img), [rootfs.bin](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/blob/main/3-Main-SoC-Realtek-RTL8196E/33-Rootfs/rootfs.bin) a [userdata.bin](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/blob/main/3-Main-SoC-Realtek-RTL8196E/34-Userdata/userdata.bin).. tedy snad. Proč má kernel příponu img a ostatní bin???

- [ ] zkusit vytáhnout image z lidl krámu
- [ ] mkrnout se, jestli stažená záloha odpovídá takovému rozdělení
- [ ] vyrobit soubor bin obsahující všechny výše zmíněné, vycpaný Fky
- [ ] flashnout a modlit se
