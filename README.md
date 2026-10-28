# Lab 07 - Ruuter ja kaks võrku (e-õpe)

Juhend: https://optalvik.github.io/avalused/praktikumid/praktikum_07.html
Teooria: peatükk 06 Võrgukiht (https://optalvik.github.io/avalused/teooria/06_vorgukiht.html)

Täna töötad kodus. Õpetaja ei ole kõrval, seega on kõik sammud siin README-s ja iga sammu järel on kirjas, mida peaksid nägema. Kui jääd kinni, vaata lõpus olevat tabelit "Kui midagi ei tööta".

Seni olid kõik su arvutid **ühes võrgus** ja kommutaator ajas asja ära. Täna ehitad kaks eraldi võrku ja paned nende vahele ruuteri. Saad teada, miks arvutil on vaja default gateway't ja mis juhtub, kui see puudub.

---

## Tänane plaan (90 min)

| Aeg | Mida teed |
|---|---|
| 0-20 min | Loe teooria peatükk 06 Võrgukiht. Kirjuta vihikusse: mis on default gateway (oma sõnadega) ja joonista bussijaama näide. |
| 20-40 min | Osa 1-2: ehita topoloogia, uuri ruuterit |
| 40-65 min | Osa 3-4: seadista ruuter ja arvutid, testi |
| 65-75 min | Osa 5: lõhu meelega ja vaata, mis juhtub |
| 75-90 min | Vasta küsimustele ja esita |

---

## Kuidas esitada (ainult brauseris)

1. **Failid:** vajuta **Add file → Upload files** ja lohista sisse:
   - `Praktikum7_Eesnimi_Perenimi.pkt`
   - `ip-brief.png`: ekraanipilt ruuteri käsust `show ip interface brief`, kus mõlemad liidesed on `up up` (Osa 4)
   - `ping.png`: ekraanipilt PC1 Command Promptist, kus ping PC2-le (192.168.2.10) õnnestub (Osa 4)
2. **Vastused:** ava see fail, vajuta pliiatsit ✏️, kirjuta vastused `>` märgi järele ja vajuta **Commit changes**.

Tähtaeg on Classroom 50-s.

---

## Osa 1: Topoloogia

Ava Packet Tracer ja lisa järgmised seadmed:

| Seade | Kust leiad | Nimi |
|---|---|---|
| Ruuter **2811** | Network Devices → Routers | R1 |
| 2 kommutaatorit **2960** | Network Devices → Switches | SW1, SW2 |
| 2 arvutit **PC** | End Devices | PC1, PC2 |

Ühenda need vaskkaablitega (**Copper Straight-Through**). Kui klikid seadmel, saad pordi ise valida.

```text
PC1 ──── SW1 ──── R1 ──── SW2 ──── PC2
            Fa0/0    Fa0/1
```

| Kust | Kuhu |
|---|---|
| PC1 FastEthernet0 | SW1 Fa0/1 |
| SW1 Fa0/24 | R1 **Fa0/0** |
| R1 **Fa0/1** | SW2 Fa0/24 |
| SW2 Fa0/1 | PC2 FastEthernet0 |

**Mida peaksid nägema:** kommutaatori ja arvutite vahel on tuled alguses oranžid ja lähevad kuni 30 sekundi pärast roheliseks. Ruuteri juures on tuled **punased**. See on õige! Ruuteri liidesed on vaikimisi välja lülitatud.

Salvesta fail kohe: `Praktikum7_Eesnimi_Perenimi.pkt`. Salvesta ka edaspidi iga osa järel (Ctrl+S).

---

## Osa 2: Uuri ruuterit

Kliki R1-l → vahekaart **CLI**. Kui küsitakse `Would you like to enter the initial configuration dialog? [yes/no]`, kirjuta `no` ja vajuta Enter.

```text
Router> enable
Router# show version
```

Otsi väljundist ja kirjuta allpool tabelisse:

- ruuteri mudel (rida, mis algab `Cisco 2811`);
- IOS-i versioon (rida `Cisco IOS Software ... Version ...`);
- mälu (rida, mis lõpeb `bytes of memory`).

Väljund on pikk. Kui lõpus on `--More--`, vajuta tühikut.

Nüüd vaata liideseid:

```text
Router# show ip interface brief
```

**Mida peaksid nägema:** iga liidese real on `unassigned` ja `administratively down down`.

`unassigned` tähendab, et liidesel pole IP-aadressi. `administratively down` tähendab, et liides on käsuga välja lülitatud. Kommutaatoril olid pordid kohe sees, ruuteril mitte.

---

## Osa 3: Seadista ruuter

### Nimi ja paroolid

Need on samad käsud, mida tegid kommutaatoriga praktikumis 2:

```text
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret cisco123
R1(config)# banner motd #Volitamata ligipaas keelatud!#
```

### Liides Fa0/0: esimese võrgu uks

```text
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Vork 1 (PC1)
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Mida peaksid nägema:** kohe pärast `no shutdown` ilmub teade `%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up`. Topoloogias muutuvad R1 ja SW1 vahel tuled roheliseks (võib võtta kuni 30 sekundit).

### Liides Fa0/1: teise võrgu uks

Tee ise sama teisele liidesele. Aadress on `192.168.2.1`, mask `255.255.255.0`, kirjeldus `Vork 2 (PC2)`.

💡 Kui ei mäleta, vaata ülevalt Fa0/0 plokki. Ainult liidese number ja aadress on teised.

### Salvesta

```text
R1(config-if)# end
R1# copy running-config startup-config
```

Vajuta Enter, kui küsitakse `Destination filename [startup-config]?`.

---

## Osa 4: Seadista arvutid ja testi

### Arvutite IP-seaded

Kliki PC1 → **Desktop** → **IP Configuration** → vali **Static**:

| | PC1 | PC2 |
|---|---|---|
| IPv4 Address | 192.168.1.10 | 192.168.2.10 |
| Subnet Mask | 255.255.255.0 | 255.255.255.0 |
| Default Gateway | 192.168.1.1 | 192.168.2.1 |

Pane tähele: iga arvuti gateway on **ruuteri liides tema enda võrgus**. PC1 jaoks on R1 aadress `192.168.1.1`, PC2 jaoks `192.168.2.1`. Sama ruuter, aga kaks ust.

### Kontrolli ruuterit

R1 CLI-s:

```text
R1# show ip interface brief
```

**Mida peaksid nägema:** `FastEthernet0/0` ja `FastEthernet0/1` real on IP-aadress ja olek `up up`. **Tee sellest ekraanipilt → `ip-brief.png`.**

```text
R1# show ip route
```

**Mida peaksid nägema** (väljundi lõpus):

```text
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet0/1
```

Sa ei kirjutanud ühtegi marsruuti, aga ruuter teab mõlemat võrku. Ta õppis need ise, kui sa liidestele aadressid andsid ja need sisse lülitasid.

### Ping läbi ruuteri

PC1 → Desktop → **Command Prompt**:

```text
C:\> ping 192.168.1.1
C:\> ping 192.168.2.1
C:\> ping 192.168.2.10
```

1. Esimene ping läheb PC1 enda gateway'le.
2. Teine läheb ruuteri teisele liidesele.
3. Kolmas läheb **teise võrku**, PC2-le.

**Mida peaksid nägema:** kõik kolm õnnestuvad. Kolmandas pingis võib esimene pakett kaduma minna (`Request timed out`), sest ARP otsib alles MAC-aadressi. See on normaalne. Korda pingi. **Tee ekraanipilt õnnestunud pingist PC2-le → `ping.png`.**

Nüüd vaata, mis teed pakett läks:

```text
C:\> tracert 192.168.2.10
```

Loe, mitu rida (hüpet) väljundis on ja milline aadress on esimene.

✅ **Kontrollpunkt:** sul on `ip-brief.png` ja `ping.png`, fail on salvestatud.

---

## Osa 5: Lõhu meelega

Nüüd uurid, miks gateway on vajalik.

1. Ava PC2 → Desktop → IP Configuration ja **kustuta Default Gateway** (lahter tühjaks).
2. **Ennusta enne:** kas PC1-st `ping 192.168.2.10` töötab nüüd? PC1 gateway on ju endiselt õige. Kirjuta oma ennustus küsimuse 4 alla.
3. PC1 Command Promptis: `ping 192.168.2.10`
4. Kirjuta tulemus ja selgitus küsimuse 4 alla.
5. **Pane PC2 gateway tagasi** (`192.168.2.1`) ja kontrolli, et ping töötab jälle. Salvesta fail.

💡 **Vihje selgituseks:** ping on kahesuunaline. PC1 saadab küsimuse, PC2 peab saatma vastuse. Kuhu PC2 vastuse saadab, kui ta ei tea, kus on uks teise võrku?

---

**Sinu nimi:** 

## Osa 2: Ruuteri info (`show version`)

| Mida | Väärtus |
|---|---|
| Mudel | |
| IOS versioon | |
| Mälu (bytes of memory) | |

## Küsimused

**1. Miks olid ruuteri liidesed alguses `administratively down`, aga kommutaatori pordid töötasid kohe? Mis käsk need sisse lülitab?**
> 

**2. Mis on PC1 default gateway ja miks just see aadress?**
> 

**3. Mida tähendab täht `C` käsu `show ip route` väljundis? Miks ruuter neid võrke teab, kuigi sa ei lisanud ühtegi marsruuti?**
> 

**4. Osa 5: sinu ennustus, tegelik tulemus ja selgitus. Miks ping ei töötanud, kuigi PC1 seaded olid õiged?**
> 

**5. Mitu hüpet näitas `tracert` ja mis oli esimese hüppe aadress? Miks just see?**
> 

---

## Kui midagi ei tööta

| Mida näed | Mis on viga | Mida teha |
|---|---|---|
| Liides on `administratively down` | `no shutdown` puudub | Mine liidesele (`interface fa0/0`) ja kirjuta `no shutdown` |
| Liides on `down down` (ilma "administratively") | kaabel pole ühendatud või vales pordis | Kontrolli kaableid: R1 Fa0/0 → SW1, R1 Fa0/1 → SW2 |
| `% Invalid input detected` | vale režiim või trükiviga | Vaata, mis on promptis: `R1#`, `R1(config)#` või `R1(config-if)#` |
| Ping gateway'le töötab, PC2-le mitte | PC2 gateway puudub või vale | Kontrolli PC2 IP Configuration |
| Kõik pingid ebaõnnestuvad | arvuti IP või mask vale | Kontrolli PC IP Configuration, käsk `ipconfig` |
| Esimene ping `Request timed out`, ülejäänud OK | ARP otsib MAC-aadressi | Normaalne, korda pingi |

---

## Lisaülesanne (kodus, vabatahtlik)

Lisa SW1 külge **PC3** (`192.168.1.20`, gateway `192.168.1.1`). Lülita **Simulation**-režiimi ja saada PC3-st ping PC2-le. Vaata, mis juhtub paketiga R1 juures: kas MAC-aadressid kaadris enne ja pärast ruuterit on samad? Aga IP-aadressid?
> 
