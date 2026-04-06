# Servarr Media Stack

Usine à médias automatisée basée sur Docker pour NAS. Gère les téléchargements, le renommage, les sous-titres et le seedage multi-trackers de façon entièrement automatisée.

## Architecture

```
Prowlarr (Indexeurs)
    ↓
Sonarr/Radarr (Recherche & Tri)
    ↓
qBittorrent (Téléchargement)
    ↓
Hardlinks → Plex Library
    ↓
Bazarr (Sous-titres)
    ↓
Cross-Seed (Ratio multi-trackers)
```

## Services

**Prowlarr** - Agrégateur d'indexeurs centralisé. Gère les trackers et envoie les résultats de recherche à Sonarr/Radarr.

**Sonarr** - Gestionnaire de séries TV. Détecte automatiquement les nouvelles épisodes, lance les recherches et génère les demandes de téléchargement.

**Radarr** - Gestionnaire de films. Même principe que Sonarr mais pour les films.

**qBittorrent** - Client torrent. Télécharge les contenus dans des dossiers catégorisés (`/data/torrents/{films|series|animes|cross-seed}`).

**Bazarr** - Téléchargement automatique de sous-titres. Scanne la bibliothèque importée et complète les fichiers manquants.

**FlareSolverr** - Proxy pour contourner les protections Cloudflare sur les trackers.

**Cross-Seed** - Trouve automatiquement les mêmes contenus sur d'autres trackers et les injecte dans qBittorrent sans utiliser d'espace disque supplémentaire (via hardlinks).

**Gluetun** - Client VPN. Chiffre le trafic qBittorrent et Cross-Seed.


## Flux de Données

1. **Recherche** - Sonarr/Radarr trouvent un contenu demandé
2. **Téléchargement** - qBittorrent récupère le torrent dans `/data/torrents/{type}`
3. **Détection** - Sonarr/Radarr reconnaît le fichier et le renomme via hardlink vers `/data/media/{type}`
4. **Sous-titres** - Bazarr télécharge les sous-titres automatiquement
5. **Seedage** - qBittorrent continue de seeder, Cross-Seed cherche d'autres trackers
6. **Affichage** - Plex scanne `/data/media/` et affiche la bibliothèque
