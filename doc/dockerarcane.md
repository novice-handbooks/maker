# Guida Installazione Arcane con Docker & GitHub Sync

Questa guida descrive come installare la piattaforma **Arcane** su un sistema dove Docker è già presente, utilizzando un repository GitHub privato per centralizzare la configurazione dei servizi (Infrastructure as Code).

## 1. Preparazione dell'Ambiente

Prima di iniziare, assicurati che il tuo utente abbia i permessi necessari per eseguire Docker e che `git` sia installato.

### Struttura delle Directory
Consigliamo di mantenere una struttura pulita sul server:
```bash
mkdir -p ~/arcane-setup/config
cd ~/arcane-setup
```

---

## 2. Configurazione Accesso GitHub Privato

Per permettere al server di comunicare con il tuo repository privato senza inserire ogni volta la password, utilizzeremo una **SSH Key** o un **Personal Access Token (PAT)**.

### Generazione Chiave SSH (Raccomandato)
1. Genera la chiave: `ssh-keygen -t ed25519 -C "server-arcane"`
2. Copia la chiave pubblica: `cat ~/.ssh/id_ed25519.pub`
3. Aggiungila su GitHub: **Settings > SSH and GPG keys > New SSH Key**.

---

## 3. Clonazione del Repository di Configurazione

Il repository privato conterrà i file `.env`, `docker-compose.yml` e le configurazioni specifiche dei servizi.

```bash
# Sostituisci con il tuo URL SSH
git clone git@github.com:tuo-utente/tuo-repo-arcane-config.git .
```



---

## 4. Installazione di Arcane via Docker

Arcane viene solitamente distribuito come immagine Docker. Creeremo un file `docker-compose.yml` che punta alle configurazioni sincronizzate dal tuo repo.

### Esempio di `docker-compose.yml`
```yaml
services:
  arcane:
    image: arcane/arcane:latest # Sostituisci con l'immagine ufficiale corretta
    container_name: arcane-server
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - ./config:/app/config
      - ./data:/app/data
    env_file:
      - .env
```

### Passaggi operativi:
1.  **Crea il file `.env`**: Inserisci le tue API key e le credenziali del database (questo file dovrebbe essere nel `.gitignore` del tuo repo, usa un `.env.example` per tracciarne la struttura).
2.  **Avvia il servizio**:
    ```bash
    docker compose up -d
    ```

---

## 5. Sincronizzazione dei Servizi e Automazione

Per fare in modo che Arcane utilizzi le configurazioni presenti nel repository GitHub per gestire gli altri servizi del server:

### Passo A: Definizione dei Servizi
Nel tuo repository GitHub, crea una cartella `services/`. Ogni sottocartella conterrà la configurazione di un servizio specifico (es. Nginx, database, bot).

### Passo B: Script di Aggiornamento (Sync)
Crea uno script `update_services.sh` nel server per automatizzare il pull delle modifiche:

```bash
#!/bin/bash
echo "Sincronizzazione con GitHub..."
git pull origin main

echo "Riavvio dei container aggiornati..."
docker compose up -d --remove-orphans

echo "Configurazione Arcane aggiornata correttamente."
```
Rendi lo script eseguibile: `chmod +x update_services.sh`.

---

## 6. Workflow di Lavoro Tipico

1.  **Locale**: Modifichi la configurazione di un servizio o aggiungi un nuovo modulo al file `docker-compose.yml` sul tuo PC.
2.  **Git**: Esegui `git commit` e `git push` verso il repository privato.
3.  **Server**: Esegui lo script `./update_services.sh` (o configura un **Webhook** per farlo automaticamente).

---

## 7. Troubleshooting Comuni

* **Permessi SSH**: Se il clone fallisce, verifica con `ssh -T git@github.com`.
* **Volumi Docker**: Assicurati che i percorsi definiti nel `docker-compose.yml` corrispondano alle cartelle create dal repository.
* **Conflitti Git**: Se modifichi file direttamente sul server, il `git pull` potrebbe fallire. Usa sempre il repository come "fonte della verità".

---