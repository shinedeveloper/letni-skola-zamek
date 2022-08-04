Chytrý zvonek + schránka s vlajkou
==================================

Cílem je vytvořit prototyp domovního zvonku pro smart home, včetně kamery.

Veškerá komunikace přes internet probíhá přes cloudový MQTT server (broker) HiveMQ, který je zdarma:

| Název             | Hodnota                                             |
|-------------------|-----------------------------------------------------|
| Host              | d57a0d1c39d54550b147b58411d86743.s2.eu.hivemq.cloud |
| Port (Web Socket) | 8884                                                |
| SSL               | Ano                                                 |
| Username          | robot                                               |   
| Heslo             | P@ssW0rd!                                           |


Čísla GPIO na Raspberry PI:

![RPi Pinout](Raspberry-Pi-pinout.png)



Máte k dispozici
----------------

-   2x Reproduktor

    Reproduktor dokáže přehrát zvuk zvonění.
    Ovládá se přes MQTT topic `/smart-doorbell/sound/1` nebo `/smart-doorbell/sound/2`,
    podle toho, ke kterému zvukovému výstupu je reproduktor připojen.

    Tedy možné topicy + zpráva (kterou je třeba do nich zaslat):
    -   `/smart-doorbell/sound/1` ← `ring`
    -   `/smart-doorbell/sound/2` ← `ring`

    První reproduktor se připojuje na audio jack 3.5" na Raspberry Pi.
    Druhý reproduktor se připojuje do USB zvukové karty SYBA.


-   Sada 2 reproduktorů a mikrofonů

    Určeno pro domovní telefon, například ve výškovém domě.

    Komunikace se dá ustavit odesláním zprávy na topic:
    -   `/smart-doorbell/sound` ← `phone-call-start`

    K ukončení komunikace dojde buď automaticky po 30 sekundách (lze do budoucna nakonfigurovat)
    nebo odesláním zprávy na topic:
    -   `/smart-doorbell/sound` ← `phone-call-stop`

    Reproduktory se zapojují dle bodu výše, první mikrofon se zapojuje do USB konektory na Raspberry Pi,
    druhý konektor se zapojuje do USB zvukové karty SYBA.


-   Klasická tlačítka (až 5 kusů)

    Ovládají se přes MQTT topic `/smart-doorbell/button/ČÍSLO_PINU_NA_RASPBERRY_PI`, kam je tlačítko připojeno.

    Možné piny: `9`, `10`, `22`, `27`, `17`

    Tedy možné topicy + zpráva (která z nich příchází):
    -   `/smart-doorbell/button/9` → `pressed`
    -   `/smart-doorbell/button/10` → `pressed`
    -   `/smart-doorbell/button/22` → `pressed`
    -   `/smart-doorbell/button/27` → `pressed`
    -   `/smart-doorbell/button/17` → `pressed`
    -   `/smart-doorbell/button/9` → `released`
    -   `/smart-doorbell/button/10` → `released`
    -   `/smart-doorbell/button/22` → `released`
    -   `/smart-doorbell/button/27` → `released`
    -   `/smart-doorbell/button/17` → `released`


-   Kamera pro domovní video telefon.

    Vyfocení fotky: 
    -   `/smart-doorbell/photo` ← `snapshot`
    -   `/smart-doorbell/photo` ← `snapshot-vflip`
    
    Varianta `-vflip` generuje obrázek vertikálně otočený.

    Výsledná fotka je uploadována na cloudovou službu ImgBB.com a její adresa je odeslána do topicu:
    -   `/smart-doorbell/photo/taken` → `https://imgbb.com/i/3256987265.jpg`

- Pravidelné focení fotek po určitou dobu
    -   `/smart-doorbell/photo` ← `recording-start`
    -   `/smart-doorbell/photo` ← `recording-start-vflip`
    -   `/smart-doorbell/photo` ← `recording-stop`

  Varianta `-vflip` generuje obrázek vertikálně otočený.

  Výsledné fotky jsou pravidelně uploadovány na cloudovou službu ImgBB.com a jejich adresa je pokaždé odeslána do topicu:
      -   `/smart-doorbell/photo/taken` → `https://imgbb.com/i/156584235.jpg`

- Rozsvícení LED
    -   `/smart-doorbell/led/7` ← `on`
    -   `/smart-doorbell/led/8` ← `on`
    -   `/smart-doorbell/led/25` ← `on`
    -   `/smart-doorbell/led/24` ← `on`
    -   `/smart-doorbell/led/23` ← `on`
    -   `/smart-doorbell/led/7` ← `off`
    -   `/smart-doorbell/led/8` ← `off`
    -   `/smart-doorbell/led/25` ← `off`
    -   `/smart-doorbell/led/24` ← `off`
    -   `/smart-doorbell/led/23` ← `off`

- Blikání LED
    -   `/smart-doorbell/led/7` ← `blink`
    -   `/smart-doorbell/led/8` ← `blink`
    -   `/smart-doorbell/led/25` ← `blink`
    -   `/smart-doorbell/led/24` ← `blink`
    -   `/smart-doorbell/led/23` ← `blink`
    -   `/smart-doorbell/led/7` ← `stop`
    -   `/smart-doorbell/led/8` ← `stop`
    -   `/smart-doorbell/led/25` ← `stop`
    -   `/smart-doorbell/led/24` ← `stop`
    -   `/smart-doorbell/led/23` ← `stop`

-   Pohyb servo motorem

    Motorem lze pohybovat odesláním hodnoty úhlu (číslo ve stupních) na správný topic:
    -   `/smart-doorbell/servo/12` ← `90`
    -   `/smart-doorbell/servo/13` ← `270`


-   Zjištění obsazenosti schránky

    Pokud pošťák vloží balíček do schránky, je třeba na to umět detekovat.
    Asi nejpraktičtější způsob detekce je senzor vzdálenosti od překážky.
    Tedy buď od druhé stěny schránky nebo od balíčku, když bude vložen.
    Ve schránce lze nainstalovat tento senzor a zjistit, jak daleko od druhé stěny 
    je nainstalovaný. Tato hodnota se při vložení balíčku do schránky
    významně změní (zmenší).
    
    Zjištění aktuální vzdálenosti od stěny / balíčku:
    -   `/smart-doorbell/distance` ← `measure-now`
    
    Senzor zpátky odešle vzdálenost v cm. Např 15,56621 cm:
    -   `/smart-doorbell/distance/measured` → `15.56621`

    Je možné si zapnout notifikaci, která se vyvolá,
    když se před senzorem ocitne překážka
    blíže než `X` cm. Například `15,678`.
    
    Poznámka: Testuje se v 5sekundových intervalech.
    -   `/smart-doorbell/distance` ← `notify-when-shorter-then(15.678)`
