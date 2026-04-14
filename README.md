# 🌱 Smart Plant Vase

> Vaso intelligente per la cura automatizzata delle piante, progettato per utenti senza esperienza nel giardinaggio. Il sistema integra monitoraggio ambientale in tempo reale, irrigazione automatica e notifiche remote tramite Telegram.

![Status](https://img.shields.io/badge/status-in%20sviluppo-yellow)
![Platform](https://img.shields.io/badge/platform-ESP32-blue)
![License](https://img.shields.io/badge/license-MIT-green)

-----

## Indice

- [Panoramica del progetto](#panoramica-del-progetto)
- [Architettura del sistema](#architettura-del-sistema)
- [Hardware](#hardware)
- [Software stack](#software-stack)
- [Struttura del repository](#struttura-del-repository)
- [Setup firmware](#setup-firmware)
- [Setup server](#setup-server)
- [Roadmap](#roadmap)
- [Licenza](#licenza)

-----

## Panoramica del progetto

Smart Plant Vase è un sistema IoT embedded che automatizza la cura delle piante attraverso il monitoraggio continuo dei parametri ambientali (umidità del suolo, temperatura, umidità dell’aria, luminosità e livello dell’acqua) e l’attivazione automatica di una pompa di irrigazione in risposta alle condizioni rilevate.

Il dispositivo comunica via Wi-Fi utilizzando il protocollo MQTT. I dati vengono acquisiti da un broker Mosquitto, elaborati tramite Node-RED e archiviati in InfluxDB per la visualizzazione su dashboard Grafana. Le notifiche operative vengono inviate all’utente tramite Telegram Bot.

**Casi d’uso principali:**

- Irrigazione automatica basata su soglie di umidità configurabili
- Monitoraggio remoto dei parametri ambientali
- Notifiche in tempo reale su anomalie o livello d’acqua basso
- Storicizzazione e visualizzazione dei dati su dashboard

-----

## Architettura del sistema

```
┌─────────────────────────────────────────────────────┐
│                    DISPOSITIVO                       │
│                                                     │
│  [Sensori] ──► [ESP32] ──► MQTT Publish             │
│  [Pompa]  ◄─── [ESP32] ◄── MQTT Subscribe           │
└─────────────────────────────────────────────────────┘
                        │
                    Wi-Fi / MQTT
                        │
┌─────────────────────────────────────────────────────┐
│                   SERVER STACK                       │
│                                                     │
│  Mosquitto ──► Node-RED ──► InfluxDB ──► Grafana    │
│                   │                                  │
│              Telegram Bot                            │
└─────────────────────────────────────────────────────┘
```

**Layout meccanico del vaso (dall’alto verso il basso):**

```
┌──────────────────┐
│   Zona suolo     │  ← Sensori: umidità, temperatura, luce
├──────────────────┤
│  Compartimento   │  ← ESP32, MOSFET driver, alimentazione
│  elettronico     │
├──────────────────┤
│  Serbatoio       │  ← Acqua, pompa 5V, sensore HC-SR04
│  acqua           │
└──────────────────┘
```

> **Principio di sicurezza critico:** L’elettronica è posizionata sopra il serbatoio dell’acqua. In caso di perdite, il liquido defluisce verso il basso, lontano dai componenti elettronici.

-----

## Hardware

### Componenti principali

|Componente                      |Modello              |Funzione                                      |
|--------------------------------|---------------------|----------------------------------------------|
|Microcontrollore                |ESP32 (38-pin)       |Unità di controllo principale, Wi-Fi integrato|
|Sensore umidità suolo           |Capacitive v1.2      |Misura umidità del substrato                  |
|Sensore temperatura/umidità aria|AHT21 / DHT22        |Monitoraggio parametri ambientali             |
|Sensore luminosità              |BH1750               |Misura intensità luminosa (lux)               |
|Sensore livello acqua           |HC-SR04 (ultrasonico)|Misura livello del serbatoio                  |
|Pompa                           |Mini pompa 5V        |Erogazione acqua al suolo                     |
|Driver pompa                    |MOSFET / Modulo relay|Protezione GPIO ESP32                         |

### Note di cablaggio

- Il driver MOSFET (o relay module) è necessario per isolare il GPIO dell’ESP32 dalla corrente assorbita dalla pompa. Il GPIO fornisce il segnale di controllo, non alimenta direttamente la pompa.
- Il serbatoio interno della scocca è rivestito con resina epossidica per impermeabilizzazione.
- La scocca è stampata in 3D con filamento PETG, scelto per la resistenza termica e chimica superiore rispetto al PLA.

-----

## Software stack

|Componente    |Tecnologia      |Ruolo                                             |
|--------------|----------------|--------------------------------------------------|
|Firmware      |C++ / PlatformIO|Acquisizione sensori, controllo pompa, MQTT client|
|Message broker|Mosquitto (MQTT)|Instradamento messaggi ESP32 ↔ server             |
|Flow engine   |Node-RED        |Logica di automazione e orchestrazione            |
|Database      |InfluxDB        |Archiviazione time-series dei dati sensoriali     |
|Dashboard     |Grafana         |Visualizzazione storica e real-time               |
|Notifiche     |Telegram Bot API|Alert e comandi remoti                            |
|Deployment    |Docker Compose  |Containerizzazione dell’intero stack server       |

-----

## Struttura del repository

```
smart-plant-vase/
├── firmware/
│   ├── src/
│   │   ├── main.cpp            # Entry point firmware
│   │   ├── config.h            # Configurazione (pin, soglie, template)
│   │   ├── sensors/            # Driver sensori (moisture, AHT21, BH1750, HC-SR04)
│   │   ├── actuators/          # Controllo pompa
│   │   ├── mqtt/               # Client MQTT
│   │   └── utils/              # Utilities (logging, timing)
│   └── platformio.ini          # Configurazione PlatformIO
│
├── software/
│   ├── node-red/
│   │   └── flows.json          # Flussi Node-RED esportati
│   ├── influxdb/
│   │   └── init.sql            # Schema inizializzazione database
│   ├── grafana/
│   │   └── dashboards/         # Dashboard JSON esportate
│   └── telegram-bot/
│       └── bot.py              # Handler comandi Telegram
│
├── hardware/
│   ├── schematics/             # Schemi elettrici (KiCad / Fritzing)
│   ├── 3d-models/              # File STL/STEP scocca e supporti
│   └── bom.csv                 # Bill of Materials con fornitori e costi
│
├── docs/
│   ├── architecture.md         # Documentazione architettura dettagliata
│   └── wiring-diagram.png      # Schema di cablaggio visivo
│
├── scripts/
│   ├── setup.sh                # Script inizializzazione ambiente
│   └── deploy.sh               # Script deploy stack server
│
├── docker-compose.yml          # Stack server completo
├── .gitignore
└── README.md
```

-----

## Setup firmware

### Prerequisiti

- [PlatformIO IDE](https://platformio.org/) (estensione VS Code o CLI)
- Driver USB-to-Serial per ESP32 (CP2102 o CH340)

### Procedura

```bash
# 1. Clona il repository
git clone https://github.com/I1Nic/smart-plant-vase.git
cd smart-plant-vase

# 2. Copia il template di configurazione e inserisci le credenziali
cp firmware/src/config.h.template firmware/src/config.h
# Modifica config.h con SSID, password Wi-Fi e parametri MQTT

# 3. Compila e carica sul dispositivo
cd firmware
pio run --target upload

# 4. Monitora l'output seriale
pio device monitor --baud 115200
```

### File `config.h` (template)

```cpp
// Wi-Fi
#define WIFI_SSID       "YOUR_SSID"
#define WIFI_PASSWORD   "YOUR_PASSWORD"

// MQTT
#define MQTT_BROKER     "192.168.1.100"
#define MQTT_PORT       1883
#define MQTT_USER       "YOUR_MQTT_USER"
#define MQTT_PASSWORD   "YOUR_MQTT_PASSWORD"

// Soglie operative
#define MOISTURE_THRESHOLD_LOW   30   // % sotto cui attivare la pompa
#define MOISTURE_THRESHOLD_HIGH  70   // % sopra cui fermare la pompa
#define PUMP_DURATION_MS         3000 // durata singola erogazione (ms)
#define SENSOR_INTERVAL_MS       60000 // intervallo lettura sensori (ms)
```

> ⚠️ Il file `config.h` è escluso dal `.gitignore`. Non committare mai credenziali nel repository.

-----

## Setup server

### Prerequisiti

- [Docker](https://docs.docker.com/get-docker/) e [Docker Compose](https://docs.docker.com/compose/)

### Avvio stack completo

```bash
# Dalla root del repository
docker-compose up -d

# Verifica che tutti i container siano attivi
docker-compose ps
```

### Accesso ai servizi

|Servizio   |URL                  |Credenziali default       |
|-----------|---------------------|--------------------------|
|Grafana    |http://localhost:3000|admin / admin             |
|InfluxDB UI|http://localhost:8086|Configurato al primo avvio|
|Node-RED   |http://localhost:1880|Nessuna (configurabile)   |
|Mosquitto  |localhost:1883       |Vedi `mosquitto.conf`     |


> ⚠️ Cambiare le credenziali default prima di esporre i servizi su reti pubbliche.

-----

## Roadmap

- [x] Definizione architettura hardware e software
- [x] Selezione e validazione componenti
- [ ] Prototipo su breadboard
- [ ] Sviluppo firmware base (acquisizione sensori + MQTT)
- [ ] Configurazione stack server (Mosquitto + Node-RED + InfluxDB + Grafana)
- [ ] Integrazione Telegram Bot
- [ ] Progettazione scocca 3D
- [ ] Assemblaggio prototipo fisico
- [ ] Test e calibrazione sensori
- [ ] Revisione PCB custom *(fase futura)*

-----

## Licenza

Distribuito sotto licenza MIT. Vedere il file <LICENSE> per i dettagli.
