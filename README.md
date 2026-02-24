# Pokus o ohnutí levné ZigBee brány Tesla z Alzy

Nedávno mi kolega přinesl ZigBee bránu Tesla, kterou koupil na Alza za dvě kilča. Ukázalo se, že jde nejspíš o stejný HW jako ZigBee brána Silvecrest Lidl. Pokusím sem se tedy nasadit stejný postup pro její ohnutí na čistě lokální přístup, jako bránu pro Home Assistanta (ZHA, nebo ZigBee2MQTT).

Tady je původní [postup pro Lidl Silvercrest](<Lidl (Tuya) SmartHome Gateway Ohýbání.md>). Ten byl celkem úspěšný, až na pár běžných zádrhelů, které tam jsou popsané.

No a tady jsou poznámky k [podobnému pokusu na bráně Tesla](<ZigBee gateway Tuya Tesla.md>). Zatím to není hotové, ale mohlo by se zdát, že do tobře dopadne.

V repozitáři jsou kompletní kopie firmwarů a postupy včetně hesel. Je to proto, že jestli ta zařízení někdy půjdou do provozu, ta hesla už dávno platná nebudou. Jako u té brány z Lidlu. A pro teď je fajn, že můžeme pracovat na skutečných datech.

V adresáři [lidl](lidl) jsou skriptíky a prográmky postahované z internetu během dobývání brány Silvecrest. Zdroje jsou v příslušném postupu.

V adresáři [tesla](tesla) je zabalený image flashky, včetně kontrolního součtu. Rozbalit a zkontrolovat se to dá skriptíkem [unpack.sh](tesla/unpack.sh). Další postup bude asi mrknout se, zda struktura těch dat zhruba odpovídá očekávaným partišnám, a nahrání nového obsahu z repozitáře zmiňovaného v [druhém odkazu](<ZigBee gateway Tuya Tesla.md>)..
