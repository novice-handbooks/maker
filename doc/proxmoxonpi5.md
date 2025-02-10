# Installa Proxmox 8 su Rasperry Pi 5

Normalmente Proxmox viene installato tramite immagine dedicata. Non essendo prevista ancora una immagine
ufficiale dedicata al Rapberry Pi 5, si procede con l'installazione nativa su Raspberry OS.

La procedura, in alcuni momenti, prevede l'accesso diretto alla scheda Pi 5.
Si consiglia di utilizzare il sistema operativo Raspberry OS in versione lite (no desktop) ed installato
su supporto SSD NVMe.

Le procedure a seguire danno per scontato:

- sistema Raspberry OS 64bit lite
- utente con accesso a comando `sudo``

Come documentazione di riferimento si consiglia il repository ufficiale:
[Proxmox-Port](https://github.com/jiangcuo/Proxmox-Port/wiki/Install-Proxmox-VE-on-Debian-bookworm)

## Preparazione del sistema

Inanzi tutto è buona norma effettuare un **aggiornamento dei pacchetti**

```shell
sudo apt update
sudo apt upgrade
```

Occorre poi **creare la password per l'utente root**

```shell
$ sudo su
$ passwd
Nuova password:
Reimmetere la nuova password:
passwd: password aggiornata correttamente
```

## Predisposizione all'installazione di Proxmox

Rimanendo loggati come root, scarichiamo le chiavi pubbliche dal repository del porting di Proxmox e
le salviamo nel percorso di sistema `/usr/share/keyrings`

```shell
curl -L https://mirrors.apqa.cn/proxmox/debian/pveport.gpg | tee /usr/share/keyrings/pveport.gpg
```

Non preoccuparsi se a terminale vengono visualizzati caratteri illeggibili. Al termine è possibile ripulire
il terminale con il comando `ctrl+L`. Eventuali caratteri rimasti non influiscono sul funzionamento.

Successivamente inseriamo nella lista dei repository dei pacchetti debian anche quello del porting di Proxmox.

```shell
echo "deb [deb=arm64 signed-by=/usr/share/keyrings/pveport.gpg] https://mirrors.apqa.cn/proxmox/debian/pve bookworm port" | tee /etc/apt/sources.list.d/pveport.list
```

Ora che abbiamo aggiunto il repository, facciamo si che `apt` lo prenda in considerazione.

```shell
$ apt update
Trovato:1 http://deb.debian.org/debian bookworm InRelease
Trovato:2 http://deb.debian.org/debian-security bookworm-security InRelease
Trovato:3 http://deb.debian.org/debian bookworm-updates InRelease
Trovato:4 http://archive.raspberrypi.com/debian bookworm InRelease
Scaricamento di:5 https://mirrors.apqa.cn/proxmox/debian/pve bookworm InRelease [3.997 B]
Scaricamento di:6 https://mirrors.apqa.cn/proxmox/debian/pve bookworm/port arm64 Packages [566 kB]
Recuperati 570 kB in 3s (179 kB/s)
Lettura elenco dei pacchetti... Fatto
Generazione albero delle dipendenze... Fatto
Lettura informazioni sullo stato... Fatto
Tutti i pacchetti sono aggiornati.
W: Acquisizione del file "port/binary-armhf/Packages" saltata in quanto il repository "https://mirrors.apqa.cn/proxmox/debian/pve bookworm InRelease" non sembra fornire tale file (voce in sources.list errata?)
```

Non preoccuparsi del messaggio di warning.

Ora occorre aggiornare la ditribuzione

```shell
apt dist-upgrade
```

Potrebbe aggiornare qualche pacchetto. Nel mio caso, avendo già effettuato l'upgrade ad inizio procedura, non sono
stati effettuati aggiornamenti.

Prima della prossimo passo occorre segnarsi l'indirizzo IP del dispositivo

```shell
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 2c:cf:67:25:e3:50 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.180/24 brd 192.168.1.255 scope global dynamic noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::cb72:7680:a684:d070/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 2c:cf:67:25:e3:51 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.181/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 57673sec preferred_lft 57673sec
    inet6 2001:b07:5d33:12c5:47ad:a67d:87aa:f672/64 scope global noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::4703:d3cd:4352:ca46/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Nel mio caso sono attive due interfacce: `eth0` e `wlan0` rispettivamente agli indirizzi
`192.168.1.180` e `192.168.1.181`

Ora occorre aggiornare il file `hosts` tramite comando `nano /etc/hosts` e modificarlo per renderlo simile a quanto segue:

```text
127.0.0.1       localhost
127.0.1.1       raspi5
192.168.1.180   raspi5

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

In cui `raspi5` è l'_hostname_ del dispositivo.

Ora si configurano le interfacce di rete per renderle utilizzabili da Proxmox.
Utilizzare il comando `nano /etc/network/interfaces`

```text
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
#source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

iface eth0 inet static

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.180/24
        gateway 192.168.1.254
        bridge-ports eth0
        bridge-stp off
        bridge-fd 0
```

> **NOTA** accertarsi che l'indirizzo del gateway sia quello corretto, altrimenti sarà impossibile accedere ad internet.

Ora installiamo le utility per i bridge di rete

```shell
apt install ifupdown2 bridge-utils
```

Se tutto è andato a buon fine la rete è stata riavviata (se si era connessi da remoto si è persa la connessione).

Per verificare che tutto funzioni provare ad effettuare un `ping www.google.com`

Come ultimo passo di predisposizione, importante affinché funzionino correttamente gli script per la
compilazione di immagini docker, occorre utilizzare kernel con pagesize 4K.

> **NOTA**
> le linee seguenti fanno riferimento alla posizione dei file `/boot/firmware` relativa a _Debian 12 Bullseye_
> nel caso la propria installazione sia basata su _Debian 11 i file sono in `/boot`
> E' possibile verificare la propria distribuzione con il comando `cat /etc/os-release`

Per questo effettuare le seguenti modifiche: editare il file `config.txt` : `nano /boot/firmware/config.txt`
ed aggiungere in calce al file la linea:

```text
kernel=kernel8.img
```

Inoltre occorre modificare la linea di comando : `nano /boot/firmware/cmdline.txt`

inserendo i seguenti parametri alla fine della linea:

```text
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

RIAVVIARE LA SCHEDA

## Installazione di Proxmox

Effettuati tutti i preparativi possiamo finalmente installare Proxmox VE.

```shell
apt install pve-edk2-firmware
```

```shell
apt install proxmox-ve postfix open-iscsi pve-edk2-firmware-aarch64
```

Durante l'installazione verranno richieste alcune informazioni, di seguito le risposte che ho selezionato:

| Richiesta | Scelta effettuata |
| --- | --- |
| Postfix Configuration | Nessuna Configurazione |
| File di configurazione pveport.list | N : mantiene la versione attualmente installata |

Ora occorre riavviare ed il server Proxmox dovrebbe partire automaticamente.

```shell
reboot
```

## Configurazione di Proxmox VE 8

Accedere all'interfaccia web all'indirizzo : [https://192.168.1.180:8006](https://192.168.1.180:8006) oppure
[https://raspi5.local:8006](https://raspi5.local:8006)

## -- TODOs - argomenti da aggiungere ed espandere

### A. allargare la dimensione dello SWAP file

   `sudo dphys-swapfile swapoff`

   edit /etc/dphys-swapfile

   ```text
   CONF_SWAPSIZE=8000

   CONF_MAXSWAP=8000
   ```

   Riavviare servizio

   ```text
   sudo dphys-swapfile setup
   sudo dphys-swapfile swapon
   ```

   verificare `sudo swapon --show`

### 1. gestire i backup delle VM e delle CT

- shared Storage
   aggiunta di storage NFS per l'archiviazione esterna dei backup
   1. Preparare la condivisione NFS su Synology vedere [qui](https://www.youtube.com/watch?v=EMQdN6W_y1Y)
   2. `Datacenter/Storage/Add: NFS`
   3. Configurare

      ```text
      ID: SynoNFS
      Server: 192.168.1.250
      Export: /volume1/proxmox
      Content: Disk image, ISO image, Container template, VZDump backup file, Container
      ```

      --> `Add`

- attivare la cache locale per la creazione del backup
   `nano /etc/vzdump.conf`

   ```text
   # vzdump default settings
   tmpdir: /tmp
   #dumpdir: DIR
   #storage: STORAGE_ID
   ...
   ```

### 2. rendere visibili le VM/CT come [hostname].local sulla propria rete locale

- install `avahi-daemon` package

### 3. [Pimox scripts](https://pimox-scripts.com/)

   Proxmox arm64 Install Scripts

### 4. Accorgimenti per far funzionare le VM arm64 anche desktop mode

- bios : OVMF (UEFI)
- initial DVD:SCSI / HD : SATA
- then NO DVD / HD (detach + reattach as SCSI0)
- Boot order (always to check)
- (some time with no display)
- if hangs on STOP or SHUTDOWN use "Monitor" then QUIT

### 5. Aggiustare DNS all'interno di CT (basate su debian 12)

- verifica dns : `cat /etc/resolv.conf`
- modifica il file con `nameserver 8.8.8.8`
- riavvia il servizio `sudo systemctl restart systemd-resolved`

### 6. montare un disco esterno su proxmox (SMB)

Distinguiamo due differenti metodi per montare un disco: il primo più "storico" è mediante la configurazione del file `fstab`,
l'altra, più "moderna" utilizza `systemd`. 

1. creare la cartella per il punto di mount: `mkdir /media/wdbackup`
2. preparare un file con le credenziali di accesso alla risorsa condivisa : `nano /etc/win-credentials`

   ```text
   username=root
   password=pass
   ```
  
3. editare il file `/etc/fstab` ed aggiungere per esempio per una connessione SMB/cifs:

   ```text
   //192.168.1.249/WDBackup /media/wdbackup cifs relatime,credentials=/etc/win-credentials,file_mode=0777,dir_mode=0777,x-systemd.automount 0 0
   ```

   questo dovrebbe montare in /media/wdbackup la condivisione WDBackup dal server 192.268.1.249, nota che in questo caso
   seppur la condivisione fosse senza guest viene comunque usato un fake user e password

In automatico `systemd` utilizza il file `fstab` per crearsi la configurazione
di mount alla partenza. Vengono creati servizi con come _mount-point.mount_. 
Per l'esempio seguente viene creato il servizio `media-wdbackup.mount`

Quindi è possibile verificare lo stato del sistema con il comando:

```shell
systemctl status media-wdbackup.mount
```

Sono possibili anche tutti i comandi disponibili con `systemctl` come `start`, `stop`, `restart` ...


### 7. condividere cartelle o mount da Proxmox host verso CT

- per effettuare la condivisione di cartella (bindmount) eseguire il comando:

   ```bash
   pct set 504 -mp0 /media/wdbackup,mp=/media/wdbackup
   ```

   in questo caso viene condivisa la cartella /media/wdbackup sulla CT in /media/wdbackup con is mp0.
   ATTENZIONE: la CT deve essere definita "priviledged" altrimenti è impossibile utilizzare la cartella in scrittura.

### 8. convertire una CT da unpriviledged to priviledged o viceversa

   Effettuare backup della CT
   Effettuare il restore scegliendo l'opzione scelta.

### 9. Creare una partizione LVM-Thin partendo da un disco full ext4

   Se si dispone di aree di SSD non partizionate allora è possibile procedere alla creazione di una partizione LVM

- usare `fdisk` per creare la nuova partizione di tipo LVM

   Si procede alla creazione della nuova partizione sfruttando
   lo spazio liberato.

   `fdisk /dev/nvme0n1` per entrare nell'utility di formattazione

   `F` per visualizzare gli spazi non partizionati. Potrebbero
   essere listati più spazi in quanto spesso per questioni di allineamento
   rimangono anche piccoli spazi disco non partizionabili.
   Appuntarsi i valori di _Start_ ed _End_ dello spazio che
   si deve utilizzare.

   `n` creare la nuova partizione: tipo _primary_, numero della
   partizione e start e end appuntati precedentemente.

   `t` per cambiare il formato della partizione in `8e` LVM

   `w` scrive le modifiche su disco (con `q` si esce senza modificare)

- creazione del "Thinpool"
   Da interfaccia web, selezionare la voce `Disks\LVM-Thin` del nodo.
   Selezionare la voce `Create:Thinpool` e da popup configurare:
      - Disk: selezionata in automatico la partizione formattata in LVM
      - Name: local-lvm
      - Add Storage: selezionato

<!-- END -->
