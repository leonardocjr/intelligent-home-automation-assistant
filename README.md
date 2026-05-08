# Intelligent Home Automation Assistant

[![DOI](https://zenodo.org/badge/1232849683.svg)](https://doi.org/10.5281/zenodo.20084254)

An intelligent virtual assistant for domestic home automation control, built around Home Assistant, Frigate, MQTT, and Docker.

## Overview

This repository contains the source configuration and deployment files for a local smart-home assistant focused on event-driven automation, video analytics, contextual monitoring, and voice-oriented workflows.

## Repository structure

```text
.
├─ docker/
│  └─ docker-compose.yml
├─ homeassistant/
│  └─ config/
├─ frigate/
│  └─ config/
└─ mosquitto/
   ├─ config/
   ├─ data/
   └─ log/
```

## Core services

- Home Assistant: automation hub and state orchestration
- Frigate: video analytics and event generation
- Mosquitto: MQTT event broker

## Additional components used in the project

### HA-Bridge

HA-Bridge is not part of the Docker Compose stack in this repository.

In the implemented environment, HA-Bridge was executed manually as a Java application from the terminal. This approach was used because Alexa discovery worked through the emulated Hue bridge exposed by HA-Bridge, while the project environment presented practical discovery limitations with other approaches.

HA-Bridge acts as an intermediate layer between Alexa routines and Home Assistant webhooks:

1. The user speaks a phrase such as “Where is the dog?”
2. Alexa matches that phrase to a routine
3. The routine turns on a virtual lamp discovered from HA-Bridge
4. HA-Bridge sends an HTTP request to a Home Assistant webhook
5. Home Assistant runs the corresponding script
6. Alexa speaks the generated response through Alexa Media Player

Typical virtual devices used in the project included:

- `DogStatus`
- `CatStatus`
- `LeonardoStatus`
- `BeatrizStatus`
- `DailyStatus`

A representative HA-Bridge startup command is:

```bash
sudo java \
  -Dserver.port=80 \
  -Daddress=0.0.0.0 \
  -Dupnp.strict=false \
  -Dupnp.config.address=YOUR_IP \
  -Dupnp.config.discovery.port=1901 \
  -jar ha-bridge-5.4.1.jar
```

### MediaMTX

MediaMTX does not need to be stored as a dedicated folder in this repository.

In this project, it was used as an auxiliary RTSP media server to publish webcam feeds or local video files as synthetic camera streams for controlled testing. It can be started independently only when needed for stream simulation or restreaming.

## Requirements

- Docker
- Docker Compose
- Linux or Windows with WSL2
- Java Runtime Environment for HA-Bridge
- Optional: Google Coral USB Accelerator
- Optional: RTSP cameras or local video sources

## Local installation

### 1. Clone the repository

```bash
git clone https://github.com/SEU-USUARIO/intelligent-home-automation-assistant.git
cd intelligent-home-automation-assistant
```

### 2. Prepare the directories

```bash
mkdir -p mosquitto/config mosquitto/data mosquitto/log
mkdir -p frigate/config frigate/media
mkdir -p homeassistant/config
mkdir -p docker
```

### 3. Create the Mosquitto password file

```bash
docker run --rm -it eclipse-mosquitto:2 mosquitto_passwd -c /tmp/passwd frigate
```

Copy the generated content to:

```text
mosquitto/config/passwd
```

### 4. Review and adjust the configuration files

Check and update:

- `docker/docker-compose.yml`
- `frigate/config/config.yml`
- `homeassistant/config/configuration.yaml`
- `homeassistant/config/automations.yaml`
- `homeassistant/config/scripts.yaml`
- `homeassistant/config/secrets.yaml.example`
- `mosquitto/config/mosquitto.conf`

Main values to adjust:

- MQTT username and password
- RTSP stream URLs
- Frigate camera definitions
- Home Assistant secrets and integrations
- time zone and local deployment settings

### 5. Start the Docker stack

```bash
docker compose -f docker/docker-compose.yml up -d
```

### 6. Check container status

```bash
docker ps
```

Expected services:

- `homeassistant`
- `frigate`
- `mosquitto`

## How to run HA-Bridge

Place the HA-Bridge `.jar` file in any local directory outside the repository if you prefer.

Start it manually from the terminal:

```bash
sudo java \
  -Dserver.port=80 \
  -Daddress=0.0.0.0 \
  -Dupnp.strict=false \
  -Dupnp.config.address=YOUR_IP \
  -Dupnp.config.discovery.port=1901 \
  -jar ha-bridge-5.4.1.jar
```

Then:

1. Open the HA-Bridge web interface
2. Configure the bridge discovery settings
3. Create one virtual lamp per query action
4. Map each lamp ON URL to the corresponding Home Assistant webhook
5. Discover devices in the Alexa app
6. Create Alexa routines that turn on the corresponding virtual lamp

Representative webhook mappings:

- `DogStatus` → `http://YOUR_IP:8123/api/webhook/dogstatustts`
- `CatStatus` → `http://YOUR_IP:8123/api/webhook/catstatustts`
- `LeonardoStatus` → `http://YOUR_IP:8123/api/webhook/leonardostatustts`
- `BeatrizStatus` → `http://YOUR_IP:8123/api/webhook/beatrizstatustts`
- `DailyStatus` → `http://YOUR_IP:8123/api/webhook/dailystatustts`

## How to test

### Infrastructure test

1. Open Home Assistant at `http://localhost:8123`
2. Open Frigate at `http://localhost:5000`
3. Confirm Mosquitto is running
4. Confirm Frigate can reach the configured camera streams

### Functional test

Validate representative scenarios such as:

- person detection
- pet detection
- vehicle detection
- MQTT event flow from Frigate to Home Assistant
- Home Assistant automations triggered by Frigate events
- last-seen logic for tracked entities
- webhook-triggered TTS responses through Alexa and HA-Bridge

### Optional Coral test

If using a Google Coral USB device:

- confirm the device is mapped to the Frigate container
- confirm Frigate starts with hardware acceleration available
- compare behavior with and without acceleration enabled

## Reproducibility

This repository is intended to preserve the software artifact and deployment setup of the project in a versioned and reproducible way.

## Citation

Please cite this repository using the metadata provided in `CITATION.cff`.

## License

This repository is released under the MIT License. See `LICENSE` for details.
