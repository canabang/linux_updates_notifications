# 🐧 Linux Updates Notifications (Home Assistant)

![Aperçu Dashboard Home Assistant](images/dashbord_HA_LUD.gif)

Ce package gère les notifications centralisées pour le suivi et les mises à jour des serveurs Linux de l'infrastructure, en synergie parfaite avec l'outil de scan [linux-update-dashboard](https://github.com/TheDuffman85/linux-update-dashboard).

Il est conçu pour offrir des performances maximales côté Home Assistant tout en assurant un monitoring exhaustif.

> **Note :** Pour l'installation de l'application tierce en elle-même, veuillez vous référer au README du GitHub officiel (bien qu'un modèle Docker soit proposé plus bas dans cette page).

## 📦 Les Automations

1. **`automation_linux_alerte_critique.yaml`** : Notification **instantanée** si un serveur détecte de nouvelles mises à jour (surtout liées à la sécurité), s'il nécessite un redémarrage, ou s'il perd soudainement la connexion au réseau.
   > **Note :** Utilise un déclencheur `state` défini explicitement par sécurité. Il faut lister une à une ses entités serveurs en début de fichier pour garantir une performance optimale du moteur Home Assistant (et garder l'outil de traçage rapide).

2. **`automation_linux_bilan_hebdo.yaml`** : Diffuse un **Rapport Global (Digest)** de tout le parc informatique (ex: tous les Vendredis à 20h15).
   > **Note :** Détecte dynamiquement tous les serveurs grâce à un template Jinja2 via la fonction `expand()`. Inclut également l'alerting lorsqu'une nouvelle version de l'application Dashboard elle-même est disponible (image Docker).

Les deux automatisations intègrent des variables prêtes à l'emploi (`notif_titre`, `generated_message`) afin d'exporter les pings très facilement sur Telegram, Discord, ou l'Application Compagnon HA.

---

## 🖥️ Carte Dashboard (Lovelace)

Retrouvez le code d'une carte optimisée (Mushroom + Auto-Entities) clé-en-main dans le fichier **[`dashboard_card.yaml`](dashboard_card.yaml)**. 
Cette carte liste dynamiquement les serveurs et offre une vue claire pour suivre et installer les mises à jour depuis l'interface native de Home Assistant.

---

## 🐳 Configuration Docker Recommendée (linux-update-dashboard)

![Interface web native du Dashboard](images/dashboard-LUD.png)

Voici le `docker-compose.yaml` minimal et vital, avec les bonnes variables pour Home Assistant :

```yaml
services:
  dashboard:
    image: ghcr.io/theduffman85/linux-update-dashboard:latest
    container_name: linux-update-dashboard
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - dashboard_data:/data
    environment:
      # Configurations Basiques
      - TZ=Europe/Paris # /!\ Indispensable pour l'alignement des CRON/Rapports
      - LUDASH_ENCRYPTION_KEY="VOTRE_SECURE_KEY"
      - LUDASH_DB_PATH=/data/dashboard.db
      - NODE_ENV=production
      
      # Configuration Home Assistant / Proxy Extérieur
      # Permet à HA de rapatrier localement l'URL de votre icône "entity_picture"
      - LUDASH_BASE_URL=http://<IP_LOCAL_DE_LA_MACHINE>:3001
      
      # - LUDASH_TRUST_PROXY=true # DÉCOMMENTER SI DANS NGINX PROXY MANAGER (NPM)

volumes:
  dashboard_data:
```

---

## ⚙️ Configuration de l'Interface Web (MQTT)

Afin que le Discovery MQTT de Home Assistant remonte sans couac l'ensemble de votre parc sous une seule intégration, il faut se rendre dans les instances de la WebUI (*Settings -> Notifications -> Add Channel -> MQTT*).

Voici les réglages clés :

![Configuration MQTT - Partie 1](images/mqtt_settings_1.png)

![Configuration MQTT - Partie 2](images/mqtt_settings_2.png)
