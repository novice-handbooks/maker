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



