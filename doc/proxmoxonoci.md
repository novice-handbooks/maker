# Installa Proxmox VE 8 su Oracle OCI (free tier)

Oracle nella piattaforma _Oracle Cloud Infrastructure_ (OCI) mette a dispisizione un ottimo _tier gratuito_
In questo _tier_ sono disponibili risorse per creare instanze di macchine virtuali nel piano che viene indicato come _always-free_.

In particolare si hanno a disposizione:
- due instanze AMD x86_64 con 1/8 OCPU (corrispondenti a 2 vCPU) e 1GB di RAM.
- una istanza ARM-based 4 core e 24 GB di RAM

Inoltre si hanno a disposizione 200GB di blocco dati da utilizzare come memoria di massa. Siccome il minimo utilizzabile per ogni istanza è pari a 49GB, alla fine si consiglia di utilizzare:
- 100GB per la istanza ARM-Based
- 50GB per ogni istanza AMD

Fatta questa premessa è chiaro che se si intende utilizzare una istanza OCI per far girare un server Proxmox VE allora l'unica
possibilità è quella di utilizza l'istanza ARM-based che presenta una buona quantità di memoria RAM, risorsa indispensabile per il sistema Proxmox VE.

Essendo questa una macchina con architettura ARM buona parte della configurazione sarà simile a quanto già rappresentato nel documento:
[Installa Proxmox 8 su Raspberry Pi 5](proxmoxonpi5.md).

La vera difficoltà è quella di riuscire ad installare l'immagine di
Proxmox VE sull'istanza OCI, inquanto è una immagine non prevista dal
sistema automatizzato messo a disposizione da Oracle.

> **Bibliografia**
>
> Buona parte delle procedure di seguito elencate sono state descritte
> sul [Frank Ruan's Blog](https://frank-ruan.com) ed in particolare 
> nell'articolo: [Installing Proxmox VE on OCI](https://frank-ruan.com/2023/03/18/installing-proxmox-ve-on-oci/)

## Debian su istanza VM.Standard.A1.Flex

Si da qui per scontato che sia stata creata una instanza selezionando l'architettura ARM-based e collegando un blocco dati
di 100GB.

Occorre segnarsi l'indirizzo IP pubblico dell'istanza ed avere accesso alla console di controllo.
Inoltre occorre avere l'accesso _ssh_ remoto, normalmente viene utilizzato una chiave pubblica fornita in fase di creazione dell'istanza.

Ora occorre assicurarsi di avere settato il firewall per permettere al traffico internet di raggiungere l'istanza.

Connettersi all'istanza tramite connessioe SSH ed scaricare l'immagine necessaria per installare il da rete Proxmox:

```shell
sudo -i
cd /boot/efi
wget https://boot.netboot.xyz/ipxe/netboot.xyz-arm64.efi
```

Una volta eseguito il download sconnettersi dalla macchina. Ora dovremo far ripartire la macchina cercando di intercettare al boot in modo da utilizzare l'immagine appena scaricata.

Nella pagina di interfaccia di OCI assicurarsi di avere lanciato la Console collegata all'istanza. Per fare questo, una volta selezionata l'istanza, scegliere la voce _Concole connection_ presente nel menù laterale.

La qui utilizzare "Lauch Cloud Shell connection". Questo apre a fondo pagina una console (Cloud Shell) connessa all'istanza.

Seguire attendamente le seguenti operazioni:

1. dalla pagina detaggli operare sul comando _Reboot_ e scegliere l'opzione 'Force reboot the instance by immediately powering off, then powering back on'

2. Fare attenzione a ciò che avviene nella console (Cloud Shell) e premere `ESC` quando inizia il reboot per far apparire la configurazione del Bios.
![Oci Bios](./images/oci-bios.png)

3. Utilizzando le frecce per muoversi selezionare: `Boot Maintenance Manager` -> `Boot From File` -> Scegli il file dall'hard disk `netboot.xyz-arm64.efi`

4. Se tutto è andato come da copione si presenta l'interfaccia iPXE seguente
![iPXE Boot](./images/ipxe-boot.png)

5. Seleziona `Linux Network Installs` -> `Debian` -> `Debian 12.0 (bookworm)` -> `Text Based Install`

6. procedere con l'istallazione facendo attenzione nel momento in cui chiede il partizionamento: scegliere 'Guided - use entire disk and set up LVM'
![iPXE partition](./images/ipxe-partition.png)

7. A seguire accettare tutte le impostazioni di default

8. Riavviare il sistema

## Configurazione di Debian

Una volta installato, il sistema ha la necessità di alcune configurazioni utili per essere facilmente utilizzato.

1. Connessione SSH con utente (non root) creato durante l'installazione utilizzando l'indirizzo pubblico della istanza.

2. installare 'sudo' e vari altri software utili
   ```shell
   su -
   Password: (inserire la password di root)
   apt update
   apt install sudo wget curl iftop vnstat neofetch vim nano net-tools
   exit
   ```

3. aggiungere l'utente standard ai 'sudoers'. Lavorando sempre come 'root' eseguire
   ```shell
   su -
   usermod -aG sudo [nome utente]
   groups [nomeutente]
   exit
   ```
   L'utente deve uscire e rientrare affinchè venga caricata l'appartenenza al gruppo 'sudo'

4. (Opzionale) abilitare sudo senza password
   ```shell
   sudo visudo
   ```
   Editare il file aperto, cercare la linea con `%sudo ...` e modificarla in 
   ```text
   %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
   ```
   Chiudere il file con `Ctrl+x`, `y`, `Invio`

5. (Opzionale) aggiungere chiave pubblica per connessione SSH senza necessita di inserimento password.

   Dal proprio computer eseguire il comando:
   ```shell
   ssh-copy-id -i .ssh/[chiave]] [user]@[ip-address]
   ```
   Viene chiesta la password e al termine dovrebbe indicare di aver copiato una chiave

## Configurazione della rete

Si andrà a configurare la rete con IP statico.

Segnarsi l'indirizzo IP della macchina virtuale letto tramite il comando `ip address`

Nel mio caso : `10.0.0.177`

### Impostazione IP statico

Editare il file `/etc/network/interfaces`

Il file si presenta così:
```text
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp
```

Occorre modificare la parte finale in :

```text
# The primary network interface
allow-hotplug enp0s3
# iface enp0s3 inet dhcp
# Define Static IP
iface enp0s3 inet static
	address 10.0.0.117
	netmask 255.255.0.0
	gateway 10.0.0.1
```

### Edit file /etc/hosts

Editare il file `/etc/hosts` che dovrebbe essere simile a:

```text
127.0.0.1       localhost
127.0.1.1      pve-oci.vcn01191127.oraclevcn.com       pve-oci

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Modificarlo in (sostituire HOSTNAME con il proprio hostname, nel mio caso `pve-oci`):

```text
127.0.0.1       localhost
PUBLIC_IP    HOSTNAME.proxmox.com   HOSTNAME
```

Riavviare l'istanza.


