# Nexus RMM

Plateforme de gestion et supervision IT à distance (Remote Monitoring & Management).

## Prérequis

- **Docker Engine** 24+ et **Docker Compose** v2
- **2 Go RAM** minimum (4 Go recommandé)
- **10 Go** d'espace disque
- Un **nom de domaine** pointant vers votre serveur
- Un **reverse proxy** (Traefik, Nginx, Caddy) pour le HTTPS

## Installation rapide

```bash
# 1. Cloner ce dépôt
git clone https://github.com/yblis/nexus-deploy.git
cd nexus-deploy

# 2. Configurer l'environnement
cp .env.example .env
nano .env  # Adaptez les valeurs (mots de passe, domaine, etc.)

# 3. Démarrer
docker compose pull
docker compose up -d
```

## Premier accès

1. Ouvrez `http://VOTRE_IP:3333` (ou votre domaine si reverse proxy configuré)
2. Créez le compte **Super Admin** au premier lancement
3. C'est prêt !

## Configuration HTTPS (recommandé)

Nexus écoute en HTTP sur le port 3333. Placez un reverse proxy devant pour le HTTPS.

**Exemple Traefik** (labels à ajouter au service `nexus-web`) :

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.nexus.rule=Host(`nexus.example.com`)"
  - "traefik.http.routers.nexus.entrypoints=websecure"
  - "traefik.http.routers.nexus.tls.certresolver=letsencrypt"
```

**Exemple Nginx** :

```nginx
server {
    listen 443 ssl;
    server_name nexus.example.com;

    ssl_certificate     /etc/ssl/nexus.crt;
    ssl_certificate_key /etc/ssl/nexus.key;

    location / {
        proxy_pass http://127.0.0.1:3333;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support (terminal, chat)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## Mise à jour

```bash
docker compose pull
docker compose up -d
```

Les données (base de données, fichiers) sont conservées dans des volumes Docker persistants.

## Licence

Nexus fonctionne en mode **Community** sans licence (25 agents, 3 techniciens).

Pour débloquer davantage d'agents et de fonctionnalités, uploadez un fichier de licence via **Administration > Licence** dans l'interface.

| Tier | Agents | Techniciens | Fonctionnalités |
|---|---|---|---|
| Community | 25 | 3 | Core |
| Starter | 100 | 10 | + Scripts, Patch, Alerting |
| Business | 250 | 25 | + Ticketing, ITAM, Rapports |
| Enterprise | 500 | 50 | + SSO, M365, Politiques |
| Enterprise+ | 1 000 | 100 | Toutes |

Contactez-nous pour obtenir une licence : **contact@example.com**

## Sauvegarde

```bash
# Sauvegarde de la base de données
docker exec nexus-db pg_dump -U nexus nexus > backup_$(date +%Y%m%d).sql

# Restauration
docker exec -i nexus-db psql -U nexus nexus < backup_20260313.sql
```

## Ports utilisés

| Port | Service | Description |
|---|---|---|
| 3333 | nexus-web | Dashboard (HTTP) |
| 8090 | nexus-api | Connexion agents (interne) |

> **Note** : Le port 8090 est utilisé en interne par les agents pour communiquer avec l'API. Si vous utilisez un reverse proxy, configurez un sous-chemin ou un sous-domaine pour `/api`.

