# Pi-hole

[_documento editato il 2025-03-16_]

## Pi-hole: il DNS fatto in casa per proteggerci

Pi-hole è una soluzione open source che funge da server DNS locale con capacità di filtraggio,
progettato principalmente per bloccare pubblicità e malware su reti domestiche o aziendali.
Opera come un "filtro" tra gli utenti della rete e il mondo internet, intercettando le
richieste DNS e confrontandole con liste nere di domini noti per contenere pubblicità
invasiva, spyware, ransomware, malware, cryptoware, cryptominer e altri tipi di contenuti indesiderati.
Installato su un dispositivo dedicato (come Raspberry Pi), può proteggere tutti i dispositivi connessi alla rete,
bloccando automaticamente le richieste a domini dannosi o pubblicitari senza necessità di configurazione su ciascun dispositivo.

## Configurazione e personalizzazione di Pi-hole

Per configurare Pi-hole, è necessario installarlo su un dispositivo che fungerà da server DNS,
come Raspberry Pi, utilizzando guide specifiche disponibili sul sito ufficiale di Pi-hole.
Una volta installato, l'utente può accedere all’interfaccia web di Pi-hole per personalizzare
le impostazioni, inclusa la gestione delle liste nere e whitelist, il monitoraggio del
traffico DNS e le statistiche sulla rete.

Nel nostro caso verrà installato come container LXC in proxmox.
E' installato utilizzando il seguente script: [https://pimox-scripts.com/scripts?id=pihole](https://pimox-scripts.com/scripts?id=pihole)
Conviene avere l'accortezza di installarlo con IP Statico (nel mio caso 192.168.1.1) per facilitarne la configurazione.

Si può accedere all'interfaccia di configurazione puntando all'indirizzo : [http://192.168.1.1/admin](http://192.168.1.1/admin)

> _Impostazione della password_
>
> Occorre creare la password di accesso prima di poter accedere la prima volta:
>
> da Proxmox accedere alla _Shell_ del container appena creato.
> Eseguire il comando : `pihole -a -p` e creare la password
>

### Pi-hole come DNS su tutti i dispositivi

Affinché il tutto sia funzionante occorre impostare in tutti i dispositivi l'indirizzo di Pi-Hole come server DNS.
Siccome questa operazione potrebbe essere lunga e, per alcuni dispositivi, potrebbe non essere facilmente fattibile
si sceglie la soluzione di _attivare il servizio DHCP_ in Pi-hole, così sarà questo servizio a programmare i
dispositivi con tutti gli indirizzi necessari.

1. disabilitare il servizio DHCP dal router o da eventuale altro server
2. Dal menù `Settings/DHCP` abilitare **DHCP server enabled**
3. **Range of IP adresses to hand out** : From: `192.168.1.50` To: `192.168.1.199`
4. **Router(gateway) IP address** : Router: `192.168.1.254`

## Alcuni link di riferimento

- [https://www.navigaresenzapubblicita.org](https://www.navigaresenzapubblicita.org)
- [Guida all'installazione](https://www.navigaresenzapubblicita.org/progetto-pi-hole-il-dns-fatto-in-casa-per-proteggerci-da-domini-che-contengono-malware-spyware-ranswomware-cryptoware-cryptominer-e-bloccare-i-domini-siti-web-che-contengono-la-pubblicita/
)
- [Configurazione e personalizzazione](https://www.navigaresenzapubblicita.org/configurazione-e-personalizzazione-di-pi-hole/)
