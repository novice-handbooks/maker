## Creazione di rete virtuale

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