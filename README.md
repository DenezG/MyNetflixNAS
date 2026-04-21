# 🎬 My Netflix

Stack multimédia auto-hébergée tournant sur Ubuntu Server avec Docker.

---

## Internet

Freebox Pop avec fibre, brancher directement à la box pour maximiser la connexion (900mbs en upload et 900mbs en download) avec un cable de catégorie 6. Faire une règle 20% seeding et 80% jeelyfin pour l'upload, donwload osef. Optimiser l'OS pour la connexion. 

---

## Securité

En France y'a l'arcom qui peut mettre une ammende même si c'est rare, ils vont regardé le peer2peer donc qbittorrent, il faut donc mettre un VPN devant qbittorrent osef de jellyfin.
Le VPN doit supporter le portforward afin d'avoir un ratio sur les sites de torrents qui me permet de télécharger autant que je veux.
Aussi niveau sécurité informatique pour mon réseau et pc, il faut sécuriser qbittorrent donc un très bon mdp pour qbittorrent vu que je l'expose et une authentification au niveua de Caddy.
Ajouter fail2ban pour empêcher le brutforce de mes applications.


---

## Hardware

SSD for OS and server, HDD for medias
<img width="894" height="319" alt="image" src="https://github.com/user-attachments/assets/5443f523-c161-430e-b772-87b04b9af343" />

processor i5 for transcodage is enough
PassMark ≈ 17,000 for a smooth single 4K transcode; ≈ 5,000 for 1080p.
<img width="760" height="80" alt="image" src="https://github.com/user-attachments/assets/5056db7b-6b08-448f-ac26-26e9446bc765" />
<img width="783" height="357" alt="image" src="https://github.com/user-attachments/assets/815d8b21-7435-4be2-9d63-8548b1586cea" />


4go RAM is enough
<img width="821" height="158" alt="image" src="https://github.com/user-attachments/assets/ed8b3eae-03d8-4f8c-b276-65f50efc2dd5" />

Un HP Prodesk mini serait idéale, il consomme peu et est plus adapté qu'un pc portable et dans des prix plus abordable.


---

## Services

| Service | URL | Rôle |
|---|---|---|
| seerr | https://domaine.fr | Demandes de films/séries |
| Jellyfin | https://jellyfin.domaine.fr | Streaming multimédia |
| Sonarr | https://sonarr.domaine.fr | Gestion des séries |
| Radarr | https://radarr.domaine.fr | Gestion des films |
| Prowlarr | https://prowlarr.domaine.fr | Gestion des indexeurs |
| Qbittorrent | https://torrent.domaine.fr | Téléchargements |

---

## Prérequis

- Ubuntu Server
- Docker + Docker Compose installés
- Ports 80 et 443 ouverts sur la box/routeur
- Enregistrements DNS A sur herbegeur domaine pointant vers l'IP publique

---

## Installation

### 1. Installer Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Cloner / copier les fichiers

```bash
mkdir -p ~/nas
cd ~/nas
# Copie docker-compose.yml et Caddyfile ici
```

### 3. Créer les dossiers

```bash
# Configs des applications
mkdir -p ~/nas/config/{sonarr,radarr,prowlarr,seerr,qbittorrent,jellyfin}

# Médias (sur ton disque principal)
sudo mkdir -p /mnt/media/{films,series,downloads}

# Donner les droits
sudo chown -R $USER:$USER /mnt/media
```

### 4. Lancer la stack

```bash
cd ~/nas
docker compose up -d
```

---

## Structure des fichiers

```
/home/user/nas/
├── docker-compose.yml
├── Caddyfile
├── README.md
└── config/
    ├── sonarr/
    ├── radarr/
    ├── prowlarr/
    ├── seerr/
    ├── qbittorrent/
    └── jellyfin/

/mnt/media/
├── films/
├── series/
├── animes/
└── downloads/
```

---

## DNS IONOS

Enregistrements A à créer sur IONOS pointant vers l'IP publique :

| Hôte | Type | Valeur |
|---|---|---|
| `@` | A | `IP_PUBLIQUE` |
| `jellyfin` | A | `IP_PUBLIQUE` |
| `sonarr` | A | `IP_PUBLIQUE` |
| `radarr` | A | `IP_PUBLIQUE` |
| `prowlarr` | A | `IP_PUBLIQUE` |
| `seerr` | A | `IP_PUBLIQUE` |
| `torrent` | A | `IP_PUBLIQUE` |

---

## Commandes utiles

### Gestion de la stack

```bash
# Démarrer tous les services
docker compose up -d

# Arrêter tous les services
docker compose down

# Redémarrer un service
docker restart sonarr

# Mettre à jour tous les services
docker compose pull
docker compose up -d
```

### Logs

```bash
# Voir les logs de Caddy (SSL, erreurs)
docker logs caddy

# Suivre les logs en temps réel
docker logs -f caddy

# Logs d'un service spécifique
docker logs sonarr
```

### Monitoring

```bash
# Voir l'état de tous les containers
docker ps

# Voir la consommation RAM/CPU
docker stats
```

---

## Configuration initiale des services

### 1. Prowlarr → Sonarr/Radarr
1. Aller sur https://prowlarr.domaine.fr
2. Ajouter les indexeurs (trackers de torrents...)
3. Settings → Apps → Ajouter Sonarr et Radarr

### 2. Sonarr/Radarr → Qbittorrent
1. Aller sur https://sonarr.domaine.fr
2. Settings → Download Clients → Ajouter Qbittorrent
3. Host : `qbittorrent`, Port : `8080`

### 3. Seerr → Sonarr/Radarr
1. Aller sur https://domaine.fr
2. Connecter le compte Plex ou configurer manuellement
3. Ajouter Sonarr (host: `sonarr`, port: `8989`)
4. Ajouter Radarr (host: `radarr`, port: `7878`)

### 4. Jellyfin
1. Aller sur https://jellyfin.domaine.fr
2. Ajouter une bibliothèque Films → `/data/movies`
3. Ajouter une bibliothèque Séries → `/data/tv`
4. Ajouter une bibliothèque Animés → `/data/anime`
5. Ajouter l'extension Fanart.tv

---

## Mise à jour

```bash
cd ~/nas
docker compose pull        # Télécharge les nouvelles images
docker compose up -d       # Recrée les containers mis à jour
docker image prune -f      # Supprime les anciennes images
```