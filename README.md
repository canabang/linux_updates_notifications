# 🐧 Linux Updates Notifications (Home Assistant)

![Aperçu Dashboard Home Assistant](images/dashbord_HA_LUD.gif)

Ce package gère les notifications centralisées pour le suivi et les mises à jour des serveurs Linux de l'infrastructure, en synergie parfaite avec l'outil de scan [linux-update-dashboard](https://github.com/TheDuffman85/linux-update-dashboard).

Il est conçu pour offrir des performances maximales côté Home Assistant tout en assurant un monitoring exhaustif.

---

## 🛠️ Prérequis

Avant de copier le code, assurez-vous d'avoir :
1.  **MQTT Broker** : Installé et configuré (ex: Mosquitto).
2.  **HACS Cards** : Pour le dashboard, les cartes suivantes sont nécessaires :
    *   [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom) 🍄
    *   [Auto-Entities](https://github.com/thomasloven/lovelace-auto-entities) 📋
    *   [Card-Mod](https://github.com/thomasloven/lovelace-card-mod) ✨ (pour les bordures et ombres)

---

## 🚀 Installation Rapide

> **Note :** Pour l'installation de l'application tierce en elle-même, veuillez vous référer au [GitHub officiel de LUD](https://github.com/TheDuffman85/linux-update-dashboard) (bien qu'un modèle Docker soit proposé plus bas à titre d'exemple).

1.  **Docker** : Déployez le Dashboard LUD (voir [Docker](#-configuration-docker-exemple)).
2.  **MQTT** : Configurez le canal de notification MQTT dans la WebUI du Dashboard (voir [MQTT](#-configuration-de-linterface-web-mqtt)).
3.  **Automations** : Copiez le contenu des fichiers `automation_*.yaml` directement dans l'interface UI de Home Assistant (Paramètres -> Automatisations -> Créer -> Nouveau -> Éditeur YAML).
4.  **Dashboard** : Copiez le contenu de `dashboard_card.yaml` dans une carte "Manuel" de votre tableau de bord.

---

## 📦 Les Automations

Les fichiers YAML fournis ici sont destinés à être **copiés-collés directement dans l'interface UI** de Home Assistant. 

1.  **`automation_linux_alerte_critique.yaml`** : Notification **instantanée** si un serveur détecte de nouvelles mises à jour (surtout liées à la sécurité), s'il nécessite un redémarrage, ou s'il perd soudainement la connexion au réseau.
    > **Note :** Il faut lister une à une vos entités serveurs (`update.ludash_...`) en début de fichier pour garantir une performance optimale (le déclencheur `state` est plus efficace sur une liste explicite).

2.  **`automation_linux_bilan_hebdo.yaml`** : Diffuse un **Rapport Global (Digest)** de tout le parc informatique (ex: tous les Vendredis à 20h15).
    > **Note :** Détecte dynamiquement tous les serveurs grâce à un template Jinja2 via la fonction `expand()`. Inclut également l'alerting lorsqu'une nouvelle version de l'application Dashboard elle-même est disponible (image Docker).

💡 **Astuce Notification** : Les deux automatisations génèrent des variables prêtes à l'emploi (`notif_titre` et `generated_message`). Cela vous permet de router très facilement vos alertes vers n'importe quel canal (Telegram, Discord, Notification Mobile) en appelant simplement ces variables dans vos actions de notification personnelles.

---

## 🖥️ Carte Dashboard (Lovelace)

![Interface web native du Dashboard](images/dashboard-LUD.png)

Retrouvez le code d'une carte optimisée (Mushroom + Auto-Entities) clé-en-main dans le fichier **[`dashboard_card.yaml`](dashboard_card.yaml)**. 

Cette carte liste dynamiquement les serveurs et offre une vue claire pour suivre et installer les mises à jour depuis l'interface native de Home Assistant.

### ℹ️ Comment trouver vos noms de serveurs ?
Une fois MQTT configuré, rendez-vous dans *Paramètres -> Appareils & Services -> MQTT*. Cherchez l'appareil nommé **Linux Update Dashboard**. Toutes les entités de type `update.*` y seront listées (ex: `update.ludash_1_mon_serveur_system`).

---

## 🐳 Configuration Docker (Exemple)

Voici un `docker-compose.yaml` **à titre d'exemple**, incluant les variables clés pour Home Assistant :

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

