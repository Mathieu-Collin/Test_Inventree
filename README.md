# InvenTree - Projet Ingénieur Citoyen CESI

Système de gestion d'inventaire open-source déployé en conteneurs Docker pour une utilisation dans le cadre d'un projet ingénieur citoyen à CESI école d'ingénieur.

## 📋 Table des matières

- [À propos](#à-propos)
- [Architecture du projet](#architecture-du-projet)
- [Prérequis](#prérequis)
- [Installation et déploiement](#installation-et-déploiement)
- [Utilisation](#utilisation)
- [Structure des données](#structure-des-données)
- [Maintenance](#maintenance)

## À propos

**InvenTree** est une plateforme open-source de gestion d'inventaire flexible et extensible. Ce déploiement dockerisé facilite l'installation et la maintenance pour des projets d'ingénieur citoyen, permettant une gestion centralisée des stocks, des pièces, et de la documentation associée.

### Fonctionnalités principales

- 📦 Gestion complète de l'inventaire (pièces, stocks, emplacements)
- 🛠️ Suivi des ordres de fabrication et d'achat
- 📊 Rapports et statistiques détaillées
- 📱 Interface web responsive
- 🔌 Système de plugins extensible
- 🔐 Gestion des droits d'accès multi-utilisateurs

## Architecture du projet

Ce projet utilise une architecture **microservices conteneurisée** avec les éléments suivants :

```
┌─────────────────────────────────────────────────────────────┐
│                     Utilisateur (Web)                       |
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ (Port 80/443)
┌─────────────────────────────────────────────────────────────┐
│          Caddy (Reverse Proxy & Serveur Web)                │
│  • Proxy inverse vers InvenTree                             │
│  • Service des fichiers statiques (CSS, JS)                 │
│  • Service des médias (images, documents)                   │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ (Port 8000)
┌─────────────────────────────────────────────────────────────┐
│    InvenTree-Server (Application principale)                │
│  • API REST et interface web                                │
│  • Gestion de la logique métier                             │
│  • Initialisation de la base de données                     │
└─────────────────────────────────────────────────────────────┘
                    ▲                          ▲
                    │                          │
        ┌───────────┴──────────┬───────────────┴─────┐
        │                      │                     │
┌───────▼────────┐  ┌──────────▼──────────┐  ┌───────▼───────┐
│ PostgreSQL DB  │  │ InvenTree-Worker    │  │ Volumes       │
│ • Données      │  │ • Tâches async      │  │ Data          │
│ • Configuration│  │ • Notifications     │  │ Static        │
│ • Utilisateurs │  │ • Exports/Rapports  │  │ Media         │
└────────────────┘  └─────────────────────┘  └───────────────┘
```

### Composants détaillés

#### 1. **PostgreSQL (inventree-db)**
- **Rôle** : Base de données principale
- **Image** : `postgres:17`
- **Données persistantes** : `./inventree_data/postgres/`
- **Configuration** : Variables d'environnement `.env`

#### 2. **InvenTree Server (inventree-server)**
- **Rôle** : Application principale, API et interface web
- **Image** : `inventree/inventree:stable`
- **Port interne** : 8000
- **Volumes** : Configuration, médias, rapports
- **Commande démarrage** : Mise à jour (`invoke update`) puis serveur (`invoke server`)

#### 3. **InvenTree Worker (inventree-worker)**
- **Rôle** : Processus d'arrière-plan asynchrone
- **Image** : `inventree/inventree:stable`
- **Responsabilités** :
  - Génération des rapports et étiquettes
  - Envoi des notifications
  - Tâches programmées (sauvegarde, nettoyage)
  - Exports de données

#### 4. **Caddy (inventree-proxy)**
- **Rôle** : Proxy inverse et serveur web performant
- **Image** : `caddy:alpine`
- **Port externe** : 80 (configurable)
- **Responsabilités** :
  - Proxy vers InvenTree-Server
  - Service des fichiers statiques (optimisation performance)
  - Service des médias (images de pièces)
  - Gestion des certificats SSL/TLS
- **Configuration** : `./Caddyfile`

## Prérequis

- **Docker** (version 20.10+)
- **Docker Compose** (version 1.29+)
- **Espace disque** : Minimum 5 GB recommandé
- **Mémoire RAM** : Minimum 2 GB (4 GB recommandé)
- **Port 80** disponible (ou port alternatif configurable)

### Installation de Docker (Linux/Ubuntu)

```bash
# Installer Docker
sudo apt-get update
sudo apt-get install docker.io docker-compose

# Ajouter l'utilisateur courant au groupe docker (optionnel, évite sudo)
sudo usermod -aG docker $USER
newgrp docker
```

## Installation et déploiement

### Étape 1 : Cloner/Télécharger le projet

```bash
cd ~/Documents/FISEA4/Vie_Asso/
git clone <url-du-repository> Test_Inventree
# OU télécharger et extraire l'archive
cd Test_Inventree
```

### Étape 2 : Configurer les variables d'environnement

1. Créer un fichier `.env` en copiant le modèle :
```bash
cp .env.exemple .env
```

2. Éditer le fichier `.env` avec vos paramètres :
```bash
nano .env
# OU avec votre éditeur préféré (VS Code, vim, etc.)
```

3. **Variables essentielles à modifier** :
   - `INVENTREE_DB_PASSWORD` : Mot de passe de la BD (TRÈS IMPORTANT !)
   - `INVENTREE_ADMIN_PASSWORD` : Mot de passe administrateur
   - `INVENTREE_ADMIN_EMAIL` : Email du compte administrateur
   - `INVENTREE_SITE_URL` : URL d'accès (pour la production : `https://votre-domaine.com`)
  - `INVENTREE_WEB_PORT` : Port d'écoute (8080 pour HTTP par défaut, configurable)

### Étape 3 : Démarrer les conteneurs

```bash
# Démarrer tous les services
docker-compose up -d

# Vérifier que tous les conteneurs sont en cours d'exécution
docker-compose ps

# Voir les logs en temps réel
docker-compose logs -f inventree-server
```

### Étape 4 : Accéder à l'application

- **Interface web** : `http://localhost:8080` (ou `http://<votre-ip>:8080`)
- **Identifiant administrateur** : Défini dans `.env` (`INVENTREE_ADMIN_USER`)
- **Mot de passe** : Défini dans `.env` (`INVENTREE_ADMIN_PASSWORD`)

### Étape 5 (Optionnel) : Créer un superutilisateur supplémentaire

```bash
docker-compose run --rm inventree-server invoke superuser
```

Suivre les prompts pour entrer le nouvel identifiant et mot de passe.

## Utilisation

### Accès à la plateforme

1. Ouvrir un navigateur web
2. Accéder à `http://localhost:8080` (ou l'IP du serveur sur le port 8080)
3. Se connecter avec les identifiants définis
4. Commencer à gérer l'inventaire !

### Gestion des conteneurs

```bash
# Arrêter les services
docker-compose down

# Redémarrer les services
docker-compose restart

# Redémarrer un service spécifique
docker-compose restart inventree-server

# Voir les logs
docker-compose logs [nom-du-service]

# Accéder au shell du conteneur InvenTree
docker-compose exec inventree-server bash
```

### Sauvegarde des données

Les données sont sauvegardées dans `./inventree_data/` :

```bash
# Sauvegarde manuelle de la base de données
docker-compose exec -T inventree-db pg_dump -U inventree inventree | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz
```

### Installation de plugins

Placer les plugins dans `./inventree_data/data/plugins/` et redémarrer le serveur :

```bash
# Copier le plugin
cp mon_plugin.py ./inventree_data/data/plugins/

# Redémarrer le serveur
docker-compose restart inventree-server
```

## Structure des données

```
inventree_data/
├── data/
│   ├── config.yaml          # Configuration InvenTree
│   ├── secret_key.txt       # Clé secrète (ne pas modifier)
│   ├── plugins.txt          # Liste des plugins actifs
│   ├── backup/              # Sauvegardes automatiques
│   │   └── *.psql.bin.gz    # Sauvegardes PostgreSQL
│   ├── media/               # Fichiers médias (images des pièces)
│   │   ├── report/          # Rapports PDF générés
│   │   │   ├── label/       # Modèles d'étiquettes
│   │   │   └── report/      # Modèles de rapports
│   │   └── [autres médias]
│   ├── static/              # Fichiers statiques générés
│   │   ├── account/
│   │   ├── admin/
│   │   ├── rest_framework/
│   │   └── web/             # Interface web compilée
│   └── plugins/             # Dossier des plugins personnalisés
└── postgres/
    └── pgdb/                # Données PostgreSQL
```

### Points importants

- **Ne pas supprimer** : `secret_key.txt` (clé de chiffrement)
- **À sauvegarder régulièrement** : Contenu du dossier `backup/`
- **À modifier si nécessaire** : `config.yaml` (configuration avancée)
- **Volumes persistants** : Les données sont maintenues entre les redémarrages

## Maintenance

### Mise à jour d'InvenTree

```bash
# Mettre à jour la version dans .env
# Puis reconstruire et redémarrer
docker-compose pull
docker-compose up -d
```

### Problèmes courants

#### Les conteneurs ne démarrent pas
```bash
# Vérifier les logs
docker-compose logs

# Vérifier que les fichiers .env sont correctement configurés
cat .env
```

#### Accès refusé lors de l'accès aux volumes
```bash
# Fixer les permissions
sudo chown -R 1000:1000 ./inventree_data/
```

#### Port déjà utilisé
```bash
# Changer le port dans .env (INVENTREE_WEB_PORT)
# Puis redémarrer
docker-compose restart inventree-proxy
```

### Vérification de l'espace disque

```bash
# Afficher l'espace utilisé par Docker
docker system df

# Voir la taille des volumes
du -sh ./inventree_data/
```

## Support et documentation

- **Documentation InvenTree** : https://docs.inventree.org/
- **GitHub InvenTree** : https://github.com/inventree/inventree
- **Issues du projet** : Pour reporter des bugs/requêtes

---

**Dernière mise à jour** : Mai 2026  
**Projet** : Ingénieur Citoyen - CESI École d'Ingénieur  
**Licence** : InvenTree est sous licence AGPL-3.0
