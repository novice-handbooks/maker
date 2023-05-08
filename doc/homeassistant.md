# Home Assistant su Raspberry Pi

Qui di seguito vengono riportate le procedure per l'installazione e la configurazione di un Hub personale per la domotica.

A questo scopo si utilizzerà una scheda Raspberry Pi 3 opportunamente configurata, sulla quale viene installato l'applicativo [Home Assistant](https://www.home-assistant.io/).

Come sito di riferimento per le procedure di installazione e configurazione si consiglia [InDomus](https://indomus.it/), una comunità dedicata alla domotica personale.

## Installazione del BRIDGE/Gateway ZigBee

ZigBee, uno degli standard più adottati in domotica personale.

Per poter gestire i dispositivi ZigBee occorre dotarsi di BRIDGE/Gateway che sono in grado di tradurre le letture ed i comandi ZigBee su un protocollo su base Ethernet.

I vari produttori forniscono BRIDGE/Gateway che però sono limitati alla propria linea di prodotti. Quindi se si desidera utilizzare apparecchiature di differenti produttori ci si trova a dover installare gateway per ogni singolo produttore.

Nel mio caso, ad esempio, ho in utilizzo componenti Xiami/Aqara (di cui possiedo anche un BRIDGE), ed ho anche dispositivi ZigBee prodotti da IKEA.

Per non dovermi dotare di singoli BRIDGE delle varie marche, opto per la creazione di un Gateway/BRIDGE propietario che è in grado di interfacciarsi con dispositivi ZigBee delle svariate marche.

Tra le varie alternative, è il caso di **deCONZ**, oggetto della presente guida, componente software che consente per l’appunto di dotarsi gratuitamente di un BRIDGE/Gateway evoluto per la gestione della propria rete ZigBee: una volta installato, tale BRIDGE/Gateway espone delle API per instradare i messaggi da e per i componenti ZigBee tramite esso gestiti.

A questo scopo occorre anche dotarsi di un dispositivo _ZigBee Coordinator (antenna)_.
Io ho optato per un dispositivo USB molto consigliato dalla comunità : [ConBee II](https://www.phoscon.de/en/conbee2)

Per quanto riguarda l'installazione di **deCONZ** ho optato per l'installazione in un _container docker_.