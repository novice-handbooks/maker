# INSTALLAZIONE RASPBERRY PI
last edited on 2021-06-24

1. Scaricare _Raspberry Pi Imager_ dal sito ufficiale [Raspberry PI](http://www.raspberrypi.org/downloads)
2. Utilizzando l'applicativo scaricato, selezionare l'immagine del sistema operativo che si vuole installare: nel dubbio scegliere "Raspberry PI OS" 
   > **NOTA**: in caso si intenda utilizzare in modo HEADLESS (senza monitor e tastiera) e preconfigurare il WiFi, vedere il capitolo dedicato prima di procedere con la creazione dell'immagine. 
3. Iserire la SD-CARD sul computer e selezionarla dall'applicativo e avviare la scrittura
4. Inserire l'SD-CARD nel Raspberry PI, accendere ed aspettare fino a completo avvio

A questo punto è possibile collegare un monitor e una tastiera per poter utilizzare il sistema operativo.

## PREDISPOSIZIONE ALL'USO HEADLESS
Nel caso non si voglia, o non si possa, utilizzare tastiera, mouse e monitor, è possibile effettuare una configurazione della connessione WIFI seguendo una delle procedure seguenti.

### Metodo automatico tramite applicazione _Raspberry PI Imager_
Il nuovo applicativo _Raspberry PI Imager_ ha una funzionalità (seppur celata) che permette di:

- preconfigurare il WiFI, prelevando automaticamente la configurazione dal sistema operativo 
- abilitare la connessione SSH con login tramite
    - utente pi e password che si può impostare preventivamente
    - utente e chiave ssh configurabile già in questa fase e che viene importata direttamente da quella di default
- impostare fuso orario e layout tastiera
- impostare il nome host

Il tutto è configurabile premendo `CTRL-SHIFT-X` prima di effettuare la scrittura dell'immagine su SD-CARD

### Predisposizione manuale di WiFi e SSH (es per Raspberry PI ZERO)
Nel caso non si voglia, o non si possa, utilizzare tastiera, mouse e monitor, è possibile effettuare una configurazione della connessione WIFI seguendo la procedura seguente.

1. inserire la SD-CARD appena configurata nel proprio computer. La formattazione prevede la presenza di una partizione `boot` formattata in FAT32 e leggibile e scrivibile anche da computer con sistema operativo Windows o MacOS
2. occorre creare un file di configurazione che sarà poi installato nella cartella /etc/wpa_supplicant del SO Raspberry. Il file da creare deve chiamarsi `wpa_supplicant.conf` con il seguente contenuto:
    
    ```sh
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=IT
     
    network={
        ssid="YOURSSID"
        psk="YOURPASSWORD"
        scan_ssid=1
    }
    ```
    > __NOTA__ : sostituire `ssid` e `psk` con le credenziali della propria rete WiFi

    >__ATTENZIONE__ : non lasciare spazi attorno al simbolo `=` in questo file, il sistema non lo ammette...

3. copiare il file all'interno della partizione `boot` nell'SD-CARD. Alla prima accensione questo file sarà automaticamente spostato nella cartella dedicata ed utilizzato dal sistema operativo.
4. Purtroppo di default il sistema Raspbian non attiva il protocollo SSH, ma esiste un modo veloce per farlo: occorre creare un file nominato `SSH` (senza estensione) nella partizione `boot` della SDCARD.


## HEADLESS LOGIN : Collegarsi al Raspberry senza utilizzo di Tastiera e Monitor

E' possibile accedere alla console del sistema Raspberry da remoto senza l'uso della tastiera collegata.

Sarà necessario collegare la Ethernet del Raspberry alla propria rete locale, dotata di DHCP.
Tramite protocollo ssh si potra connettersi al dispositivo.

Seguire i seguenti passi:

1. Scoprire quale indirizzo IP è stato fornito al Raspberry. Per fare questo occorre o utilizzare un qualsiasi tool di IP Scanner di rete oppure verificare il dato dal proprio Router.

   > NOTA : di default la scheda si presenta in rete con hostname `raspberrypi` quindi potrebbe non essere necessario cercare l'IP e provare a raggiungere la scheda all'indirizzo `raspberry.local` (da MacOS o Linux)
3. Utilizzando un terminale SSH è ora possibile accedere alla console del Raspberry
    user: `pi` password: `raspberry`
    ```
    ssh {indirizzo IP | hostname}
    ```
4. Ora è possibile accedere al menù di configurazione da terminale di Raspbian:
    ```
    sudo raspi-config
    ```

## Istallazione di NODE su Raspberry PI Zero (armv6l)
Purtroppo i build ufficiali delle ultime versioni di Node non sono compilati per la piattaforma hardware ARM v6 utilizzata su Raspberry PI zero.

Per verificare la piattaforma usare il comando:

```sh
uname -m
```

1. scaricare il nodejs compilato per la piattaforma dai [Unofficial builds](https://unofficial-builds.nodejs.org/download/) del sito ufficiale di NodeJS.
Ad esempio il compilato della versione nodejs.14.17.1 per armv6 si trova [qui](https://unofficial-builds.nodejs.org/download/release/v14.17.1/node-v14.17.1-linux-armv6l.tar.xz)

   Quindi scaricare il binario tramite il seguente comando

   ```sh
   wget https://unofficial-builds.nodejs.org/download/release/v14.17.1/node-v14.17.1-linux-armv6l.tar.xz
   ```
2. scompattare l'archivio binario appena scaricato all'interno della cartella nella quale si intende installare Node, ad esempio in `/usr/local/lib/nodejs

    ```sh
    sudo mkdir -p /usr/local/lib/nodejs
    
    sudo tar -xJvf node-v14.17.1-linux-armv6l.tar.xz -C /usr/local/lib/nodejs
    ```

3. aggiungere il percorso alla variabile d'ambiente PATH. Editale il file `~/.profile` ed aggiungere:

    > *alternativamente è possibile utilizzare il metodo spiegato al punto successivo*

    ```
    export PATH=/usr/local/lib/nodejs/node-v14.17.1.linux-armv6l/bin:$PATH
    ```
     riavvia il profilo

    ```sh
    . ~/.profile
    ```

 4. creare i link simbolici

    > *metodo alternativo rispetto al punto precedente*

    __NOTA__: alternativamente al questo metodo è possibile creare dei link simbolici agli applicatidi node, npm e npx direttamente all'interno di /usr/bin. In questo modo tutti gli utenti possono avere accesso a node

    ```
    sudo ln -s /usr/local/lib/nodejs/node-v14.17.1-linux-armv6l/bin/node /usr/bin/node

    sudo ln -s /usr/local/lib/nodejs/node-v14.17.1-linux-armv6l/bin/npm /usr/bin/npm
    
    sudo ln -s /usr/local/lib/nodejs/node-v14.17.1-linux-armv6l/bin/npx /usr/bin/npx
    ```

5. test del corretto funzionamento di node, npm  e npx

    ```
    node -v
    
    npm version
    
    npx -v
    ```

---
Per ora è tutto

alv67
