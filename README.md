# Docker Compose: InfluxDB 2.7 + Mosquitto MQTT + Grafana

Dieses Projekt startet eine vollständige Zeitreihen-Datenplattform mit:

- **InfluxDB 2.7** – Zeitreihen-Datenbank
- **Mosquitto MQTT Broker** – Für IoT- und Sensordaten
- **Grafana** – Visualisierung der Daten aus InfluxDB

---

## 📁 Verzeichnisstruktur

```
.
├── docker-compose.yml
└── mosquitto
    ├── config
    │   └── mosquitto.conf
    ├── data
    └── log
```

> Stelle sicher, dass die Mosquitto-Verzeichnisse und die Konfigurationsdatei vorhanden sind.

---

## 🧾 Mosquitto Konfiguration (`mosquitto.conf`)

```conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883
allow_anonymous true
```

> Aktuell sind **anonyme Verbindungen erlaubt** (ohne Login). Für ein sicheres Setup siehe weiter unten.

---

## 🚀 Start

```bash
docker-compose up -d
```

---

## 🔗 Dienste & Zugänge

| Dienst     | URL / Port             | Zugangsdaten                 |
|------------|------------------------|------------------------------|
| InfluxDB   | http://localhost:8086  | Benutzer: `admin`, PW: `admin123` |
| Grafana    | http://localhost:3000  | Benutzer: `admin`, PW: `admin123` |
| Mosquitto  | MQTT-Port: `1883`      | aktuell ohne Login           |

> Ersetze `localhost` durch die IP der VM, wenn du nicht lokal zugreifst.

---

## 🧩 Grafana mit InfluxDB verbinden

1. Öffne Grafana unter http://localhost:3000
2. Logge dich ein mit `admin` / `admin123`
3. Navigiere zu **Configuration → Data Sources**
4. Wähle **InfluxDB**
5. Konfiguriere:

| Einstellung      | Wert                        |
|------------------|-----------------------------|
| URL              | `http://influxdb:8086`      |
| Auth Methode     | **Token**                   |
| Token            | `my-super-secret-token`     |
| Organization     | `my-org`                    |
| Default Bucket   | `my-bucket`                 |
| Version          | **Flux**                    |

---

## 🔐 (Optional) Mosquitto absichern mit Passwort

Falls du Login-Zugang für MQTT erzwingen möchtest:

### 1. `mosquitto.conf` anpassen:

```conf
allow_anonymous false
password_file /mosquitto/config/passwordfile
```

### 2. Passwortdatei erzeugen:

```bash
docker run --rm -v "$PWD/mosquitto/config:/mosquitto/config" eclipse-mosquitto \
  mosquitto_passwd -b /mosquitto/config/passwordfile mqttuser mqttpass
```

> Erstellt einen Benutzer `mqttuser` mit Passwort `mqttpass`.

### 3. Mosquitto neu starten:

```bash
docker-compose restart mosquitto
```

---

## 📡 MQTT Test

```bash
# Nachricht senden:
mosquitto_pub -h <IP> -t test/topic -m "Hallo MQTT" -u mqttuser -P mqttpass

# Nachricht empfangen:
mosquitto_sub -h <IP> -t test/topic -u mqttuser -P mqttpass
```

> Bei erlaubtem `allow_anonymous true` kannst du `-u` und `-P` weglassen.

---

## 🧹 Container stoppen

```bash
docker-compose down
```

Daten in den Volumes bleiben erhalten (InfluxDB, Grafana-Daten).

---

## 🛠️ Voraussetzungen

- Debian/Ubuntu-VM mit Docker & Docker Compose
- Netzwerkzugriff auf Ports `3000`, `8086`, `1883`

