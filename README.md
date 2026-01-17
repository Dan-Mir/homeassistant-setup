# 🏠 Home Assistant - Setup Domotica

Setup completo di Home Assistant con integrazione di dispositivi Xiaomi, Tuya, sensori custom e dashboard UI personalizzata.

## 📋 Indice

- [Descrizione](#-descrizione)
- [Struttura del Progetto](#-struttura-del-progetto)
- [Prerequisiti](#-prerequisiti)
- [Installazione](#-installazione)
- [Configurazione](#-configurazione)
- [Avvio del Sistema](#-avvio-del-sistema)
- [Componenti Custom](#-componenti-custom)
- [Dashboard UI](#-dashboard-ui)
- [Comandi Utili](#-comandi-utili)
- [Backup e Ripristino](#-backup-e-ripristino)
- [Troubleshooting](#-troubleshooting)

---

## 🎯 Descrizione

Questo repository contiene la configurazione completa di un sistema Home Assistant per la gestione domotica, includendo:

- **Home Assistant Core** - Server principale per automazione domestica
- **Integrazioni Xiaomi** - Gestione aspirapolvere e dispositivi Xiaomi
- **Integrazioni Tuya** - Controllo dispositivi Tuya
- **HACS** - Home Assistant Community Store per componenti custom
- **Text-to-Speech** - Sintesi vocale in italiano (Piper)
- **Voice Recognition** - Riconoscimento vocale (Whisper)
- **Dashboard UI Custom** - Interfaccia web personalizzata

---

## 📁 Struttura del Progetto

```
.
├── docker-compose.yml                          # Configurazione Docker per Home Assistant
├── .env                                        # Variabili d'ambiente (NON committare)
├── .env.example                                # Template per variabili d'ambiente
├── home_assistant/
│   └── config/
│       ├── configuration.yaml                  # Configurazione principale HA
│       ├── automations.yaml                    # Automazioni
│       ├── scripts.yaml                        # Script
│       ├── scenes.yaml                         # Scene
│       ├── secrets.yaml                        # Credenziali (NON committare)
│       ├── blueprints/                         # Blueprint per automazioni
│       └── custom_components/                  # Componenti custom
│           ├── hacs/                          # Home Assistant Community Store
│           ├── home_agent/                    # Agente custom
│           ├── xiaomi_cloud_map_extractor/    # Mappe aspirapolvere Xiaomi
│           └── xiaomi_miot/                   # Integrazione Xiaomi MIoT
├── Home-Assistant-custom-components-Xiaomi-Cloud-Map-Extractor/
│   ├── custom_components/                      # Componente Xiaomi Map Extractor
│   ├── blueprints/                             # Blueprint per automazioni vacuum
│   └── scripts/                                # Script di elaborazione mappe
├── UI_dashboard/
│   ├── backend/                                # Backend Node.js per dashboard
│   └── public/                                 # Frontend dashboard
├── viomi-integration-hacs/                     # Integrazione HACS per Viomi
├── piper-data/                                 # Modelli TTS italiano
│   ├── it_IT-paola-medium.onnx
│   └── it_IT-riccardo-x_low.onnx
├── whisper-data/                               # Modelli per speech-to-text
└── tuya_env/                                   # Virtual environment Python per Tuya
```

---

## 🔧 Prerequisiti

### Software Richiesto

- **Docker** >= 20.10
- **Docker Compose** >= 2.0
- **Git**
- **Python** 3.13+ (per ambiente virtuale Tuya)
- **Node.js** >= 18 (per dashboard UI)

### Hardware Consigliato

- Raspberry Pi 4 (4GB RAM minimo) o equivalente
- Connessione di rete stabile
- Dispositivi smart compatibili

---

## 🚀 Installazione

### 1. Clona il Repository

```bash
git clone https://github.com/Dan-Mir/homeassistant-setup.git
cd homeassistant-setup
```

### 2. Configura le Variabili d'Ambiente

Crea il file `.env` copiando il template:

```bash
cp .env.example .env
```

Modifica `.env` con i tuoi dati:

```bash
nano .env
```

Compila tutti i campi necessari:

```env
raspberry_hostname="tuo-hostname"
raspberry_user="tuo-user"
raspberry_password="tua-password"
raspberry_ip="192.168.1.xxx"
home_assistant_port="8123"
home_assistant_user="admin"
home_assistant_password="password-sicura"
xiaomi_home_email="tua-email@example.com"
xiaomi_home_password="password-xiaomi"
aspirapolvere_ip="192.168.1.xxx"
aspirapolvere_token="token-del-dispositivo"
aspirapolvere_nome="Robot Aspirapolvere"
MCP_Server_HA_token="token-per-mcp"
```

### 3. Configura Secrets di Home Assistant

Crea il file `home_assistant/config/secrets.yaml`:

```bash
nano home_assistant/config/secrets.yaml
```

Esempio di contenuto:

```yaml
# HTTP
http_base_url: "http://192.168.1.xxx:8123"

# Xiaomi
xiaomi_username: "tua-email@example.com"
xiaomi_password: "password-xiaomi"

# Aspirapolvere
vacuum_ip: "192.168.1.xxx"
vacuum_token: "token-dispositivo"

# Database (opzionale)
db_url: "postgresql://user:pass@localhost/homeassistant"
```

---

## 🏃 Avvio del Sistema

### Avvio Rapido

```bash
# Dalla directory root del progetto
docker-compose up -d
```

### Verifica dello Stato

```bash
# Controlla che il container sia in esecuzione
docker ps | grep home-assistant

# Visualizza i log in tempo reale
docker logs -f home-assistant
```

### Riavvio

```bash
# Riavvia Home Assistant (dopo modifiche configurazione)
docker-compose restart

# Oppure riavvia solo il container
docker restart home-assistant
```

### Accesso all'Interfaccia

Apri il browser e vai a:

```
http://localhost:8123
```

oppure

```
http://IP_DEL_TUO_RASPBERRY:8123
```

Al primo avvio dovrai:
1. Creare un account amministratore
2. Configurare nome casa e posizione
3. Completare l'onboarding

---

## ⚙️ Configurazione

### Configurazione Principale

Il file `home_assistant/config/configuration.yaml` contiene le impostazioni base:

```yaml
default_config:

# Text to speech italiano
tts:
  - platform: google_translate
    language: 'it'

# HTTP
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1

# Automazioni
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
```

### Configurazione Docker

Il file `docker-compose.yml` è già configurato con:

- **Network Mode**: `host` (accesso diretto alla rete locale)
- **Volumes**: 
  - `./home_assistant/config:/config` (configurazione persistente)
  - `/run/dbus:/run/dbus:ro` (comunicazione D-Bus)
- **Environment**: 
  - `TZ=Europe/Rome` (timezone italiano)
- **Privileged**: Abilitato per accesso hardware
- **Restart Policy**: `unless-stopped`

---

## 🧩 Componenti Custom

### 1. HACS (Home Assistant Community Store)

Gestisce l'installazione di componenti custom dalla community.

**Repository**: https://hacs.xyz/

**Componenti installati tramite HACS**:
- Xiaomi Cloud Map Extractor
- Xiaomi MIoT
- Viomi Integration

### 2. Xiaomi Cloud Map Extractor

Estrae e visualizza in tempo reale le mappe generate dall'aspirapolvere Xiaomi/Roborock.

**Features**:
- Visualizzazione mappa live
- Zone e stanze configurabili
- Cronologia pulizie
- Camera entity per Lovelace

**Configurazione**: Via interfaccia web Home Assistant → Integrazioni → Xiaomi Cloud Map Extractor

**Blueprint disponibili**:
- `disable_vacuum_camera_update_when_docked.yaml` - Disabilita aggiornamenti quando è in base
- `update_map_extractor.yaml` - Aggiornamento automatico mappe

### 3. Xiaomi MIoT

Integrazione completa per dispositivi Xiaomi usando il protocollo MIoT.

**Supporta**: Sensori, luci, aspirapolvere, condizionatori, purificatori aria

### 4. Home Agent

Componente custom personalizzato (dettagli specifici nella cartella del componente).

### 5. Viomi Integration

Integrazione specifica per aspirapolvere Viomi.

**Repository**: `viomi-integration-hacs/`

---

## 🎨 Dashboard UI

### Backend

Directory: `UI_dashboard/backend/`

```bash
cd UI_dashboard/backend

# Installa dipendenze
npm install

# Avvia in sviluppo
npm run dev

# Avvia in produzione
npm start
```

**Struttura**:
- `routers/` - API endpoints
- `services/` - Logica di business

### Frontend

Directory: `UI_dashboard/public/`

Servito dal backend o configurabile con Nginx/Apache.

---

## 🔊 Text-to-Speech (Piper)

Modelli vocali italiani pre-scaricati in `piper-data/`:

- **Paola Medium** (`it_IT-paola-medium.onnx`) - Voce femminile qualità media
- **Riccardo X-Low** (`it_IT-riccardo-x_low.onnx`) - Voce maschile qualità bassa

### Utilizzo in Automazioni

```yaml
service: tts.speak
data:
  entity_id: media_player.salotto
  message: "Buongiorno! La temperatura è 22 gradi."
  language: it
```

---

## 🎤 Speech-to-Text (Whisper)

Modelli per riconoscimento vocale in `whisper-data/`.

**Modello**: `faster-whisper-tiny` (bilanciato velocità/accuratezza)

---

## 🔌 Integrazione Tuya

Ambiente virtuale Python dedicato: `tuya_env/`

### Attivazione Ambiente

```bash
source tuya_env/bin/activate
```

### Comandi Disponibili

```bash
# Tool di gestione Tuya
tinytuya

# Scansione dispositivi
tinytuya scan

# Wizard configurazione
tinytuya wizard
```

### Deattivazione

```bash
deactivate
```

---

## 💻 Comandi Utili

### Switchare Container tra Directory

Se lavori su più branch/directory (`Domotica_main` e `Domotica_jules_branch`), usa questi comandi per spostare il container da una directory all'altra.

#### Da Domotica_main a Domotica_jules_branch

```bash
# 1. Ferma e rimuovi il container dalla directory corrente
cd /home/danym/Desktop/Domotica_main
docker stop home-assistant
docker rm home-assistant

# 2. Avvia il container dalla nuova directory
cd /home/danym/Desktop/Domotica_jules_branch
docker-compose up -d

# 3. Verifica che sia attivo
docker ps | grep home-assistant
```

#### Da Domotica_jules_branch a Domotica_main

```bash
# 1. Ferma e rimuovi il container dalla directory corrente
cd /home/danym/Desktop/Domotica_jules_branch
docker stop home-assistant
docker rm home-assistant

# 2. Avvia il container dalla nuova directory
cd /home/danym/Desktop/Domotica_main
docker-compose up -d

# 3. Verifica che sia attivo
docker ps | grep home-assistant
```

**Nota**: `docker-compose down` potrebbe non funzionare se il container è stato avviato da un'altra directory. In tal caso, usa sempre `docker stop` + `docker rm`.

---

### Docker - Home Assistant

```bash
# Avvia Home Assistant
docker-compose up -d

# Ferma Home Assistant
docker-compose down

# Riavvia Home Assistant
docker-compose restart

# Visualizza log
docker logs -f home-assistant

# Accedi alla shell del container
docker exec -it home-assistant bash

# Aggiorna l'immagine
docker-compose pull
docker-compose up -d

# Verifica salute del container
docker inspect home-assistant | grep -A 5 Health
```

### Git

```bash
# Verifica stato
git status

# Aggiungi file modificati
git add .

# Commit
git commit -m "Descrizione modifiche"

# Push su GitHub
git push origin main

# Pull modifiche remote
git pull origin main

# Verifica differenze
git diff
```

### Backup Configurazione

```bash
# Backup completo
tar -czf backup-homeassistant-$(date +%Y%m%d).tar.gz \
  home_assistant/config/*.yaml \
  home_assistant/config/.storage \
  .env \
  docker-compose.yml

# Backup solo configurazioni YAML
tar -czf backup-yaml-$(date +%Y%m%d).tar.gz \
  home_assistant/config/*.yaml
```

### Ripristino Configurazione

```bash
# Estrai backup
tar -xzf backup-homeassistant-YYYYMMDD.tar.gz

# Riavvia dopo ripristino
docker-compose restart
```

### Controllo Validità YAML

```bash
# Usa il container HA per validare
docker exec -it home-assistant \
  hass --script check_config --config /config
```

---

## 🔄 Backup e Ripristino

### Backup Automatico

Home Assistant crea backup automatici in `home_assistant/config/.storage/`.

### Backup Manuale

1. **Via Interfaccia Web**:
   - Impostazioni → Sistema → Backup
   - Crea nuovo backup
   - Scarica il file `.tar`

2. **Via Comando**:

```bash
# Crea snapshot
docker exec -it home-assistant ha backups new --name "backup-manuale"

# Lista backup disponibili
docker exec -it home-assistant ha backups list
```

### Ripristino

```bash
# Via interfaccia web
# Impostazioni → Sistema → Backup → Ripristina

# Via CLI
docker exec -it home-assistant ha backups restore <slug-backup>
```

---

## 🐛 Troubleshooting

### Home Assistant non si avvia

```bash
# Controlla i log
docker logs home-assistant

# Verifica errori di configurazione
docker exec -it home-assistant hass --script check_config
```

### Errore "Permission Denied"

```bash
# Correggi permessi directory config
sudo chown -R 1000:1000 home_assistant/config
```

### Container si riavvia continuamente

```bash
# Verifica stato
docker ps -a

# Ispeziona container
docker inspect home-assistant

# Rimuovi e ricrea
docker-compose down
docker-compose up -d
```

### Dispositivi Xiaomi non rilevati

1. Verifica credenziali in `secrets.yaml`
2. Controlla token dispositivo con:

```bash
source tuya_env/bin/activate
miio-extract-tokens
```

3. Verifica che il dispositivo sia sulla stessa rete

### Database corrotto

```bash
# Backup database esistente
cp home_assistant/config/home-assistant_v2.db \
   home_assistant/config/home-assistant_v2.db.backup

# Rimuovi database (verrà ricreato)
rm home_assistant/config/home-assistant_v2.db*

# Riavvia
docker-compose restart
```

### Reset Completo

⚠️ **ATTENZIONE**: Cancella tutte le configurazioni!

```bash
# Backup prima di procedere!
tar -czf backup-before-reset-$(date +%Y%m%d).tar.gz home_assistant/config

# Rimuovi container e dati
docker-compose down -v

# Rimuovi configurazione
rm -rf home_assistant/config/.storage

# Ricrea
docker-compose up -d
```

---

## 🔐 Sicurezza

### File Sensibili

I seguenti file **NON devono mai essere committati** su Git:

- `.env` - Variabili d'ambiente
- `home_assistant/config/secrets.yaml` - Credenziali
- `home_assistant/config/*.db*` - Database
- `home_assistant/config/*.log*` - Log
- `home_assistant/config/.storage/` - Dati runtime

Sono già protetti dal `.gitignore`.

### Backup Credenziali

Conserva una copia sicura offline di:
- File `.env`
- File `secrets.yaml`
- Token dispositivi

---

## 📚 Risorse

- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [HACS Documentation](https://hacs.xyz/docs/setup/download)
- [Xiaomi Cloud Map Extractor](https://github.com/PiotrMachowski/Home-Assistant-custom-components-Xiaomi-Cloud-Map-Extractor)
- [Community Forum](https://community.home-assistant.io/)
- [GitHub Repository](https://github.com/Dan-Mir/homeassistant-setup)

---

## 📝 Note

- **Timezone**: Configurato per `Europe/Rome`
- **Lingua**: Italiano (IT)
- **Porta predefinita**: 8123
- **Restart policy**: Container si riavvia automaticamente (eccetto stop manuale)

---

## 🤝 Contributi

Per contribuire al progetto:

1. Fork del repository
2. Crea un branch (`git checkout -b feature/nuova-funzionalita`)
3. Commit delle modifiche (`git commit -m 'Aggiunta nuova funzionalità'`)
4. Push del branch (`git push origin feature/nuova-funzionalita`)
5. Apri una Pull Request

---

## 📄 Licenza

Questo progetto è personale. Per uso dei componenti custom, fare riferimento alle rispettive licenze.

---

## 👤 Autore

**Dan-Mir**

- GitHub: [@Dan-Mir](https://github.com/Dan-Mir)
- Repository: [homeassistant-setup](https://github.com/Dan-Mir/homeassistant-setup)

---

## 🆘 Supporto

Per problemi o domande:

1. Controlla la sezione [Troubleshooting](#-troubleshooting)
2. Consulta i [log](#docker---home-assistant)
3. Apri un issue su GitHub
4. Consulta il [Community Forum](https://community.home-assistant.io/)

---

**Ultimo aggiornamento**: Gennaio 2026
