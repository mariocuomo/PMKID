# PMKID ATTACK

Questo repository è di supporto all'articolo riguardo il PMKID ATTACK disponibile [qui](https://medium.com/@mariocuomo/pmkid-attack-parte-1-9ae5433ea85e).<br>
L'articolo tra ispirazione dal [post](https://hashcat.net/forum/thread-7717.html) pubblicato il 4 agosto 2018 sul forum di **hashcat** riguardo una vulnerabilità nel protocollo Wi-Fi Protected Access (WPA e WPA2).


### SCENARIO
Un Client che non è in possesso della password di un Access Point cerca di riceve un messaggio contenente un **PMKID** - una stringa cifrata il cui unico parametro ignoto è il **PMK**.<br>
Il **PMK** è generato a partire dalla password in chiaro e altre varibili note al Client.<br>
Conoscendo il processo generativo del PMK dalla password e quello del PMKID a partre dal PMK, il Client cerca di costruire un **PMKID*** a partire da password note.


