# Alv67 Electronic Maker's FAQ

Raccolta di appunti personali su:

- schede elettroniche e la loro programmazione
- predisposizione ed installazione di un prorpio Homeserver e vari servizi collegati
- utilizzo di servizi cloud (per lo più gratuiti)

## M5Stack CORE Fire

M5Stack Core è una serie di microcontrollori modulari e compatti basati principalmente su ESP32,
progettati per rendere lo sviluppo di dispositivi IoT, interfacce utente e prototipi elettronici semplice e veloce.
È prodotta dalla società M5Stack e fa parte dell’ecosistema Core Series

### 🔧 Caratteristiche comuni

- ESP32: MCU potente con Wi-Fi + Bluetooth integrati.
- Display integrato: tipicamente 320x240 pixel.
- Slot per moduli: compatibilità con moduli, unità e accessori M5Stack, grazie a connettori standard (Grove, BUS, etc).
- Batteria LiPo: integrata e ricaricabile via USB.
- Case modulare: impilabile per aggiungere moduli GPS, batterie, sensori, relè, ecc.

### 🗒️ I miei appunti

- [Programmazione scheda con Arduino IDE](m5stack.md)

## Raspberry PI

La Raspberry Pi è una single-board computer (SBC) a basso costo e di piccole dimensioni,
sviluppata dalla Raspberry Pi Foundation con lo scopo di promuovere l’insegnamento dell’informatica e della programmazione,
ma oggi ampiamente usata anche in progetti industriali, domotici e maker.

È un mini computer completo, che può essere collegato a un monitor,
tastiera e mouse ed eseguire sistemi operativi basati su Linux (come Raspberry Pi OS)
o anche Windows IoT.
Offre una vasta gamma di porte di espansione per l’elettronica e la robotica.

### 🔧 Caratteristiche principali (Raspberry Pi 4/5, modello standard)

- Processore: Quad-core o superiori (es. ARM Cortex-A72 o Cortex-A76).
- RAM: da 1 GB fino a 8 GB (LPDDR4 o LPDDR4X).
- Storage: microSD card o SSD tramite USB/PCIe.
- Porte USB: 2x USB 2.0, 2x USB 3.0.
- HDMI: fino a 2 uscite micro-HDMI (4K supportato).
- Ethernet: fino a Gigabit.
- Wi-Fi + Bluetooth: integrati (a partire dalla Pi 3).
- GPIO: 40 pin per collegare componenti elettronici (LED, sensori, motori, ecc.).
- Alimentazione: tramite USB-C (5V 3A o più).

### 🗒️ I miei appunti

- [Installazione Raspberry Pi](RaspberryPI.md)
- [Home Assistant su Raspberry Pi](homeassistant.md)
- [Raspberry Pi 5 con NVMe](raspi5nvme.md)
- [Installa Proxmox su Raspberry Pi OS](proxmoxonpi5.md)

## Homelab & Homeserver

Un _HomeLab_ (da home laboratory) è un ambiente di test, sperimentazione o apprendimento IT realizzato a casa.
Può variare da un semplice Raspberry Pi fino a rack interi con server, switch, storage e firewall.

✳️ Obiettivi principali:

- Imparare sistemi operativi (Linux, Windows Server, BSD…)
- Sperimentare con virtualizzazione (es. Proxmox, VMware, Hyper-V)
- Gestire container (Docker, Kubernetes)
- Simulare ambienti enterprise
- Fare test prima di applicare soluzioni in produzione
- Imparare rete, sicurezza, DevOps, ecc.

Un _HomeServer_ è un server domestico usato per eseguire servizi utili nella vita di tutti i giorni.
È spesso parte di un _HomeLab_, ma può anche esistere da solo con scopi pratici.

📌 Funzioni tipiche di un _HomeServer_:

- File server / NAS: accesso remoto ai file, backup centralizzati
- Media server: Plex, Jellyfin, Emby
- Server web o app: per hosting personale
- Domotica: Home Assistant, MQTT, Zigbee2MQTT
- Pi-hole: blocco pubblicità e tracciamento a livello di rete
- VPN server: per accedere in sicurezza alla rete di casa
- Container e VM: per servizi isolati
- Git server, CI/CD, monitoraggio (Grafana, Prometheus)

Come si può ben capire tra _Homelab_ e _Homeserver_ il passo e breve e la differenza è quasi solo filosofica.

Per questo io faccio difficoltà a distinguere le due categorie, quindi gli appunti che seguono si adattano ad entrambe le categorie.

### 🗒️ I miei appunti

- [Installazione e configurazione server Pi-hole su Proxmox VE](pihole.md)

## Servizi cloud

In questo paragrafo vengono raccolti appunti, esempi pratici e indicazioni relative
all’utilizzo, creazione e configurazione di servizi cloud, con particolare attenzione alle
offerte gratuitamente accessibili tramite i piani Free Tier messi a disposizione dai
principali fornitori (come AWS, Google Cloud, Azure, Oracle Cloud e altri). L’obiettivo è
documentare esperienze e procedure utili per sperimentare e apprendere le tecnologie cloud
senza costi iniziali, esplorando servizi come macchine virtuali, database gestiti, storage,
funzioni serverless, monitoraggio e altro. Questa raccolta può servire sia come riferimento
personale che come base per futuri progetti o ambienti di test.

### 🗒️ I miei appunti

- [Oracle Cloud Interface OCI free tier](ocifreetier.md)
- [Installa Proxmox VE 8 su Oracle OCI (free tier)](proxmoxonoci.md)
