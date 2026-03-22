# 🚀 Tutoriel d'Installation : Linux Update Dashboard via Docker

Voici un guide pas à pas pour installer **Linux Update Dashboard** sur votre propre serveur à l'aide de Docker Compose. C'est la méthode recommandée pour garder le contrôle sur vos données et configurer facilement les intégrations avec Home Assistant.

## Étape 1 : Préparation de la clé de chiffrement forte

Avant toute chose, l'application a besoin d'une clé de chiffrement robuste (`LUDASH_ENCRYPTION_KEY`). C'est ce qui lui permet de stocker de façon ultra-sécurisée les identifiants et clés SSH de vos serveurs (pour la fonction "Upgrade auto").

1. Ouvrez votre terminal (ou connectez-vous au serveur hôte) et générez la clé avec cette commande native Linux :
   ```bash
   openssl rand -base64 32
   ```
2. **Copiez précieusement** la longue chaîne de caractères qui s'affiche, vous allez en avoir besoin juste après !

## Étape 2 : Création du fichier `docker-compose.yaml`

Dans votre gestionnaire de fichiers ou via Portainer, créez une nouvelle stack/fichier `docker-compose.yaml` avec la structure suivante :

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
      # --- Configurations vitales ---
      - TZ=Europe/Paris # Requis pour la cohérence des planifications
      - LUDASH_ENCRYPTION_KEY=COLLEZ_VOTRE_CLE_ICI
      - LUDASH_DB_PATH=/data/dashboard.db
      - NODE_ENV=production
      
      # --- Intégration Home Assistant (Auto-Discovery) ---
      # Permet à HA de rapatrier localement l'URL des icônes
      - LUDASH_BASE_URL=http://<IP_DU_SERVEUR_DOCKER>:3001
      
      # À décommenter UNIQUEMENT si l'application est derrière un reverse proxy pour un accès web (ex: Nginx Proxy Manager)
      # - LUDASH_TRUST_PROXY=true

volumes:
  dashboard_data:
```

### ⚠️ Précisions sur les variables d'environnement cruciales :
- **`LUDASH_ENCRYPTION_KEY`** : Collez exactement la clé cryptographique générée à l'étape 1. Sans elle, l'application refusera de s'allumer par sécurité.
- **`LUDASH_BASE_URL`** : Remplacez `<IP_DU_SERVEUR_DOCKER>` par l'IP réelle de la machine qui fait tourner ce conteneur. C'est critique pour MQTT Auto-Discovery, car Home Assistant récupère l'image du logo de l'intégration à partir de cette adresse.

## Étape 3 : Lancement de l'application

Une fois le fichier complété et sauvegardé, lancez la stack (via l'interface de Portainer, ou en ligne de commande locale) :

```bash
docker compose up -d
```

L'image est très légère. Le serveur Node.js et la base de données SQLite vont s'initialiser instantanément, en sauvegardant le tout dans le volume persistant `dashboard_data`.

## Étape 4 : Première connexion

Ouvrez un navigateur web et rendez-vous sur l'adresse du port 3001 : `http://<IP_DU_SERVEUR_DOCKER>:3001`

1. Lors de la toute première visite, l'application vous invitera obligatoirement à créer le compte Administrateur local.
2. Entrez un nom d'utilisateur et un mot de passe très robuste *(ne le perdez pas !)*.
3. Et voilà ! L'infrastructure est en place. 

Il ne vous reste plus qu'à : 
- Ajouter votre première machine via le bouton **"Add System"** (un simple login/mot de passe ou une clé SSH locale suffit).
- Configurer l'onglet **"Settings -> Notifications -> MQTT"** pour propulser l'ensemble de votre flotte dans le tableau de bord natif de Home Assistant !

---
*Ce mode opératoire respecte les préconisations de sécurité officielles du dépôt GitHub d'origine.*
