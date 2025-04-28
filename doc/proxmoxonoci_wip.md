## Creazione di rete virtuale (VEDERE PIU AVANTI)

> Attenzione !!!
> 
> Questa procedura è in fase di test. Solo al termine positivo del test verrà 
> cancellata questa annotazione. 
> 
> I passi seguenti sono da ritenersi solo indicativi.



Il primo passo da fare è assicurarsi che il sistema SDN (Software Development Network) sia attivo.

Per ogni nodo del cluster, accertarsi che al fondo del file `/etc/network/interfaces` ci sia attiva la seguente linea:

```text
source /etc/network/interfaces.d/*
```

Poi per ogni nodo occorre installare il pacchetto `dnsmasq` per attivare il servizio DHCP

```bash
apt update
apt install dnsmasq
# disable default instance
systemctl disable --now dnsmasq
```

Inoltre installare anche il pacchetto `FRRouting` necessario per configurazioni avanzate di SDN

```bash
apt install frr-pythontools
```

A questo punto è consigliabile effettuare un _reboot_ dei nodi.



Perfetto! Ecco una guida passo-passo per configurare Zone e VNet in Proxmox VE 8.1+ per far comunicare container LXC su nodi diversi, sfruttando la modalità overlay (quindi rete virtuale distribuita).

⸻

🧰 Requisiti iniziali
	•	Proxmox VE 8.1 o superiore
	•	Due nodi già in cluster
	•	LXC abilitati
	•	Pacchetti aggiornati:

apt update && apt full-upgrade



⸻

🛠️ 1. Abilita il supporto per vnets e zones

Verifica che la gestione del networking cluster sia attiva:

pveversion | grep pve-manager

Deve essere almeno 8.1.

Puoi anche abilitare l’interfaccia da Web GUI:
	•	Vai su Datacenter → Permissions → User Permissions
	•	Assegna il ruolo PVEAdmin al tuo utente se serve, per accedere alla sezione VNet/Zone.

⸻

🌐 2. Crea una Zone
	•	Vai su Datacenter → Zones 
	•	Clicca su “Add” -> VXLAN
	•	ID: default (o un nome a tua scelta)
	•	Description: es. Zona LAN virtuale
	•	Nodes: seleziona entrambi i tuoi nodi (pve1, pve2)
	•	Peer Address List: 100.x.y.z 100.a.b.c
	•	Lascia le altre opzioni di default



Clicca Create.

⸻

🔌 3. Crea una VNet in modalità Overlay
	•	Vai su Datacenter → VNets
	•	Clicca “Add”
	•	ID: lan-vnet
	•	Zone: default
	•	CIDR: 10.42.0.0/24 (esempio)
	•	Gateway: 10.42.0.1
	•	Mode: overlay ✅
	•	DNS: opzionale (puoi mettere 1.1.1.1 o lasciare vuoto)
	•	DHCP Range: 10.42.0.100 - 10.42.0.200 (se vuoi usare il DHCP)
	•	MTU: lascialo vuoto o metti 1400
	•	Nodes: entrambi i nodi

Clicca Create.

⸻

🧱 4. (Opzionale) Abilita Firewall e ACL

Vai su:
	•	Datacenter → Firewall
	•	Imposta Enable = Yes
	•	Puoi poi gestire regole per zona o container

⸻

🧪 5. Crea due LXC connessi alla VNet

Su pve1:
	•	Vai su Create CT
	•	Nella scheda Network:
	•	Aggiungi interfaccia
	•	In Bridge, seleziona la tua VNet (lan-vnet@default)
	•	Imposta IP statico (es: 10.42.0.10/24) oppure lascia vuoto per usare DHCP

Su pve2:
	•	Stessa cosa, con IP diverso (es: 10.42.0.11/24)

⸻

🧪 6. Test di rete

Dopo l’avvio dei container:

pct exec 100 ping 10.42.0.11
pct exec 101 ping 10.42.0.10

Dovresti vedere traffico attraversare i due nodi grazie al VXLAN gestito da Proxmox!

⸻

🚀 Extra: Accesso a internet da container

Se vuoi che i container abbiano anche uscita su internet:
	1.	Vai su ciascun nodo (pve1, pve2)
	2.	Abilita NAT da VNet a WAN con iptables o Proxmox Firewall:

iptables -t nat -A POSTROUTING -s 10.42.0.0/24 -o vmbr0 -j MASQUERADE



Puoi automatizzarlo con uno script o aggiungerlo a /etc/network/interfaces con post-up.

⸻

🧹 Consigli finali
	•	Le VNet overlay usano automaticamente VXLAN su UDP, e funzionano bene anche su Tailscale!
	•	Puoi avere più zone per isolare ambienti (es: zona “prod”, zona “test”)
	•	Funziona anche con VM, non solo LXC

⸻

Vuoi uno script per automatizzare la parte iptables/NAT oppure uno YAML di esempio per creare container collegati alla VNet?

   
## Preparazione di una rete privata


Sì, assolutamente sì!
Puoi creare una rete interna in Proxmox che:
	•	dia IP privati ai container LXC in automatico tramite DHCP,
	•	e faccia anche NAT per farli uscire su internet,
	•	e permetta a te di fare port forwarding verso i container.

In pratica, puoi avere una mini-“rete casalinga” virtuale dentro Proxmox.

⸻

Come fare

Ti spiego come fare passo passo 👇

⸻

1. Crea una nuova bridge (vmbr1)

> NOTA
>
> Ho effettuato la creazione dall'interfaccia Networks di Proxmox, e applicando la rete risulta attiva.
> Editando a mano il file non avviene l'attivazione in automatico.

Accedi alla tua Proxmox VM (quella su Oracle Cloud) via SSH.

Apri il file /etc/network/interfaces:

```bash
sudo nano /etc/network/interfaces
```

E aggiungi una nuova bridge chiamata vmbr1, tipo:

```text
auto vmbr1
iface vmbr1 inet static
    address 192.168.100.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

👉 Qui vmbr1 ha IP 192.168.100.1 e nessuna porta fisica collegata (bridge interno).

⸻

2. Installa un piccolo server DHCP+NAT

Ora installiamo un software che dia:
	•	DHCP ai container
	•	NAT verso Internet (uscita su IP pubblico)

Il più semplice è dnsmasq + NAT via iptables.

Installa dnsmasq:

```bash
sudo apt update
sudo apt install dnsmasq
```


⸻

3. Configura dnsmasq per vmbr1

Crea un file /etc/dnsmasq.d/lxc-net.conf:

```bash
sudo nano /etc/dnsmasq.d/lxc-net.conf
```

Contenuto:

```text
interface=vmbr1
bind-interfaces
dhcp-range=192.168.100.10,192.168.100.100,12h
```

	•	Questo fa partire DHCP sulla vmbr1
	•	Da IP .10 a .100, lease 12 ore

Poi riavvia dnsmasq:

```bash
sudo systemctl restart dnsmasq
```


⸻

4. Abilita il NAT (per far uscire i container su internet)

Imposta il NAT con iptables:

```bash
# Abilita il forwarding di pacchetti
sudo sysctl -w net.ipv4.ip_forward=1

# Regola NAT: maschera tutto ciò che esce da Internet
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o enp0s3 -j MASQUERADE
```

(Nota: sostituisci enp0s3 con il nome della tua scheda di rete pubblica, puoi vederla con ip a)

Se vuoi rendere permanente:

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```


⸻

5. Configura i container LXC per usare vmbr1

Quando crei o modifichi un container:
	•	Vai su Proxmox Web UI
	•	Scegli il container
	•	Sezione Network
	•	Imposta la scheda su Bridge: vmbr1 (invece di vmbr0 o altro)
	•	Metodo IP: DHCP

Risultato:
Ogni container attaccato a vmbr1 riceverà automaticamente un IP 192.168.100.x 🎯

⸻

6. E ora il Port Forwarding?

Con questa configurazione:
	•	Puoi fare come prima (iptables PREROUTING) per girare le porte dal IP pubblico della VM verso gli IP dei container.

Ad esempio:

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.100.10:80



⸻

Schema finale:

Internet --> Oracle VM IP Pubblico:80 --> iptables --> Container IP Privato (es. 192.168.100.10:80)

E tutti i container si gestiscono automaticamente via DHCP 🔥

⸻



## TAILSCALE come pubblicazione della sottorete

Sì, confermo! 🔥
Con Tailscale puoi pubblicare l’intera sottorete vmbr1, così che tu possa raggiungere tutti i container LXC via Tailscale, senza esporre nulla su internet.

Questa funzione si chiama Subnet Router.

⸻

Cosa significa “Subnet Router”?
	•	La macchina Proxmox (dove hai Tailscale installato)
➔ pubblicherà verso la VPN Tailscale la rete 192.168.100.0/24 (la rete dei tuoi container su vmbr1).
	•	Dal tuo computer, se sei collegato a Tailscale, potrai raggiungere direttamente tutti i container LXC.

⸻

Come configurarlo

1. Abilita il “forwarding” IP su Proxmox

Prima di tutto, assicurati che il tuo sistema permetta di inoltrare pacchetti di rete:

sudo sysctl -w net.ipv4.ip_forward=1

Per renderlo permanente (dopo un reboot), aggiungilo in /etc/sysctl.conf:

echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p



⸻

2. Rilancia Tailscale con il parametro --advertise-routes

Ora, sul nodo Proxmox dove gira Tailscale, devi pubblicare la rete:

sudo tailscale up --advertise-routes=192.168.100.0/24

(aggiungi anche --accept-routes=true se vuoi subito ricevere eventuali subnet da altri)

⸻

3. Approva il subnet router nella dashboard di Tailscale

Vai sulla dashboard Tailscale:

https://login.tailscale.com/admin/machines

	•	Cerca la tua macchina Proxmox.
	•	Vedrai una notifica “Subnet routes pending”.
	•	Clicca Approve per il route 192.168.100.0/24.

✅ Ora il route è visibile a tutti i device Tailscale autorizzati.

⸻

4. Assicurati che il client Tailscale possa usare le rotte

Se stai usando un client Tailscale:
	•	Su desktop (Windows, macOS, Linux) va tutto automatico.
	•	Su Mobile (iOS/Android) devi abilitare “Use exit node / Allow remote subnets” nelle impostazioni Tailscale.

⸻

🚀 Risultato finale

Ora puoi da qualsiasi dispositivo connesso alla tua rete Tailscale:
	•	pingare gli LXC tipo ping 192.168.100.10
	•	raggiungerli via browser su http://192.168.100.10
	•	SSH direttamente se hai server SSH sui container.

⸻

Schema visivo della rete

[ Tu su Tailscale ] ---> [ Proxmox ] ---> [ vmbr1 ] ---> [ LXC 192.168.100.x ]



⸻

Riepilogo

Step	Cosa fai
1	Abiliti IP forwarding
2	Rilanci Tailscale con --advertise-routes=192.168.100.0/24
3	Approvi route da dashboard Tailscale
4	Usi Tailscale normalmente per raggiungere i container

