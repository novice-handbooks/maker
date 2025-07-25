# Home Assistant su Raspberry Pi

[_documento editato il 2024-06-28_]

Qui di seguito vengono riportate le procedure per l'installazione e la configurazione di un Hub personale per la domotica.

A questo scopo si utilizzerà una scheda Raspberry Pi 3 opportunamente configurata,
sulla quale viene installato l'applicativo [Home Assistant](https://www.home-assistant.io/).

Come sito di riferimento per le procedure di installazione e configurazione si consiglia [InDomus](https://indomus.it/),
una comunità dedicata alla domotica personale.

## Installazione del BRIDGE/Gateway ZigBee

ZigBee, uno degli standard più adottati in domotica personale.

Per poter gestire i dispositivi ZigBee occorre dotarsi di BRIDGE/Gateway che sono in grado di tradurre le letture ed
i comandi ZigBee su un protocollo su base Ethernet.

I vari produttori forniscono BRIDGE/Gateway che però sono limitati alla propria linea di prodotti.
Quindi se si desidera utilizzare apparecchiature di differenti produttori ci si trova a
dover installare gateway per ogni singolo produttore.

Nel mio caso, ad esempio, ho in utilizzo componenti Xiami/Aqara (di cui possiedo anche un BRIDGE),
ed ho anche dispositivi ZigBee prodotti da IKEA.

Per non dovermi dotare di singoli BRIDGE delle varie marche, opto per la creazione di un Gateway/BRIDGE proprietario
che è in grado di interfacciarsi con dispositivi ZigBee delle svariate marche.

Tra le varie alternative, è il caso di **deCONZ**, oggetto della presente guida,
componente software che consente per l’appunto di dotarsi gratuitamente di un BRIDGE/Gateway
evoluto per la gestione della propria rete ZigBee: una volta installato, tale BRIDGE/Gateway
espone delle API per instradare i messaggi da e per i componenti ZigBee tramite esso gestiti.

### ZigBee Coordinator (antenna)

Per far funzionare il BRIDGE/Gateway occorre anche dotarsi di un dispositivo _ZigBee Coordinator (antenna)_.
Io ho optato per un dispositivo USB molto consigliato dalla comunità : [ConBee II](https://www.phoscon.de/en/conbee2)

Per quanto riguarda l'installazione di **deCONZ** ho optato per l'installazione in un _container docker_.

> **N.B.** Si consiglia, di utilizzare delle _mini prolunghe USB_ per collegare sulla porta USB del Raspberry Pi
l'antenna ConBee. Questo per evitare più che accertate interferenze.

#### **Aggiunta di ulteriori porte seriali**

Dato che il Raspberry dispone di due porte seriali, ma una è disabilitata di default, installando l'antenna potrebbe
smettere di funzionare il modulo Bluetooth o non essere riconosciuta l'antenna stessa.
Per evitare ciò occorre effettuare una modifica al file `config.txt` presente nella root della scheda Micro SD.

```sh
sudo nano /boot/config.txt
```

La modifica da fare prevede l’aggiunta, in fondo al file, del seguente codice (avere cura di
aggiungere una riga vuota dopo le due righe di codice):

```text
enable_uart=1
dtoverlay=pi3-disable-bt
```

uscire e salvare (`Ctrl-X`, `Y`, `invio`), e infine spegnere il Raspberry Pi con:

```sh
sudo shutdown now
```

Completato lo _shutdown_, spegnere **fisicamente** l'unità per poi riaccenderla subito dopo.

#### **Installazione del modulo ConBee II**

Dopo aver inserito il modulo d'antenna su di una porta USB ed avvere riavviato il Raspberry Pi occorre effettuare
_una verifica_ per appurare che la porta logica sia stata assegnata all'antenna dal sistema operativo.

Per fare questo prepariamo un semplice script per elencare le interfacce hardware in uso.

Collegandosi al Raspberry tramite SSH creare lo script:

```sh
sudo nano ports.sh
```

e copiare all'interno dell'editor il seguente codice:

```bash
#!/bin/bash

for sysdevpath in $(find /sys/bus/usb/devices/usb*/ -name dev); do
    (
        syspath="${sysdevpath%/dev}"
        devname="$(udevadm info -q name -p $syspath)"
        [[ "$devname" == "bus/"* ]]
        eval "$(udevadm info -q property --export -p $syspath)"
        [[ -z "$ID_SERIAL" ]]
        echo "/dev/$devname - $ID_SERIAL"
    )
done
```

uscire e salvare (`Ctrl-X`, `Y`, `invio`).
Eseguire poi i comandi:

```sh
sudo chmod 777 ports.sh
./ports.sh
```

A questo punto lo script è eseguito ottenendo un output simile al seguente:

```sh
/dev/bus/usb/001/001 - Linux_6.1.21-v8+_dwc_otg_hcd_DWC_OTG_Controller_3f980000.usb
/dev/bus/usb/001/003 - 0424_ec00
/dev/bus/usb/001/002 - 0424_9514
/dev/ttyACM0 - dresden_elektronik_ingenieurtechnik_GmbH_ConBee_II_DE2685810
/dev/bus/usb/001/005 - dresden_elektronik_ingenieurtechnik_GmbH_ConBee_II_DE2685810
```

Questo elenco riporta tutte le interfacce allocate. La prima parte è l’allocazione logica assegnata dal sistema operativo
(eg. /dev/bus/usb/001/001) mentre la seconda l’etichetta descrittiva del device.

Ovviamente bisognerà autonomamente capire quale sia quella che ci interessa utilizzare.

> **N.B.**
> Nell’esempio di cui sopra si nota una duplicazione della voce “dresden_elektronik_ingenieurtechnik_GmbH_ConBee_II_DE2685810“.
> Questo perché viene elencato sia l’indirizzo della porta al quale è fisicamente connesso il device
> USB (/dev/bus/usb/001/005) sia l’indirizzo alias logico (/dev/ttyACM0).
> Si consiglia di utilizzare sempre quest’ultimo tipo di indirizzamento piuttosto che il primo, e il motivo è semplice:
> in caso di cambio di porta, l’alias logico non cambia mai, evitando quindi problemi di configurazione
> sui software che utilizzino i device in questione.

### Instanziare deCONZ

Prerequisito principale è avere installato `docker` sul Raspberry Pi.

Si sceglie di utilizzare `docker compose` in tal modo tutta la configurazione viene registrata all'interno del file
`docker-compose.yaml` assieme agli altri container definiti (ad esempio il container di Home Asssistant).

Al file `docker-compose.yaml` aggiungere la seguente configurazione (sotto il blocco _services_):

```yaml
deconz:
    container_name: deconz
    image: deconzcommunity/deconz
    volumes:
    - "/opt/deconz:/root/.local/share/dresden-elektronik/deCONZ"
    devices:
    - "/dev/ttyXXX"
    environment:
    - "TZ=Europe/Rome"
    - "DECONZ_WEB_PORT=40850"
    - "DECONZ_VNC_MODE=1"
    - "DECONZ_VNC_PORT=40851"
    - "DECONZ_VNC_PASSWORD=mia_password"
    network_mode: host
    restart: always
```

dove ovviamente nel campo `devices` occorre indicare la porta precedentemente identificata (nell'esempio precedente `ttyACM0`)

Una volta salvato il file eseguire il comando:

```sh
docker compose up -d deconz
```

Lanciato il comando, attendere il completamente del primo avvio. Ci potrebbe volere un po’:
per leggere in tempo reale i log e verificare quindi cosa stia succedendo all’interno del container, eseguire il comando:

```sh
docker container logs deconz -f
```

in fondo al log dovremmo trovare qualcosa come:

```text
[deconzcommunity/deconz] Starting deCONZ...
[deconzcommunity/deconz] Current deCONZ version: x.xx.xx
[deconzcommunity/deconz] Web UI port: 40850
[deconzcommunity/deconz] Websockets port: 443
[deconzcommunity/deconz] VNC port: 40851
```

Congratulazioni: deCONZ è ora operativo.
