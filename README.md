# Homelab — Infrastructure personnelle

Infrastructure self-hosted tournant 24/7, pensée autour de trois axes : isolation réseau stricte, exposition publique minimale, et résilience des données.

<br>

## Architecture réseau

Le réseau repose sur un routeur de qualité (ARMv8, 10 Gbps SFP+) avec une segmentation en VLANs distincts et une politique firewall en **default-DROP** sur toutes les directions.

| Usage                                  | Accès inter-VLAN                |
|----------------------------------------|---------------------------------|
| Management (accès physique uniquement) | Accès total vers tous les VLANs |
| Homelab (serveurs)                     | Isolé, sortie WAN uniquement    |
| Réseau personnel                       | Sortie WAN + accès Homelab      |
| Réseau invité                          | Sortie WAN uniquement           |

L'accès distant à l'ensemble du réseau est géré exclusivement via **Tailscale** avec une politique ACL qui distingue les devices `client` (autorisés à se connecter) des `server` (qui n'initient aucune connexion sortante vers le mesh). Aucun port n'est ouvert directement sur le WAN.

Serveur DNS local hébergé dans le routeur, avec interception DNS globale pour l'ensemble du réseau.

<br>

## Serveurs

### Proxmox (Mini PC)

Hyperviseur principal hébergeant l'ensemble des VMs du homelab. Sauvegardes planifiées vers le NAS.

| VM        | CPU | RAM   | Rôle                                         |
|-----------|-----|-------|----------------------------------------------|
| labdocker | 4   | 16 Go | Services Docker personnels                   |
| labcode   | 4   | 8 Go  | Environnement de développement (code-server) |
| labgit    | 2   | 4 Go  | Instance Gitea (miroir GitHub)               |

**[labdocker](./docker/labdocker/README.md)** regroupe les services du quotidien :

- **[Homepage](./docker/labdocker/homepage.yml)** — dashboard personnalisé (développé maison, profils travail/perso)  
- **[Portainer](./docker/labdocker/portainer.yml)** — administration des conteneurs  
- **[Wealthfolio](./docker/labdocker/wealthfolio.yml)** — suivi d'investissements (actions, crypto)  
- **[Dozzle](./docker/labdocker/dozzle.yml)** — consultation des logs conteneurs  
- **[IT-Tools](./docker/labdocker/it-tools.yml)** — boîte à outils dev  
- **[Stirling PDF](./docker/labdocker/stirling-pdf.yml)** — édition PDF  
- **[ConvertX](./docker/labdocker/convertx.yml)** — conversion de fichiers  
- **[Gitea Mirror](./docker/labdocker/gitea-mirror.yml)** — Synchronisation automatique des repositories GitHub vers instance locale  
- **[Diun](./docker/labdocker/diun.yml)** — alertes de mise à jour des images Docker  
- **[Beszel](./docker/labdocker/beszel.yml)** - dashboard de monitoring (CPU, RAM, réseau) pour tous les serveurs physiques du homelab  

**labcode** — Environnement de développement Web accessible via navigateur.  

**labgit** — Instance Gitea pour redondance et sauvegarde des repositories.  

<br>

### [NAS](./docker/NAS/README.md)

Stockage centralisé accessible en SMB et WebDAV. Héberge également :

- **Plex** — Gestion et streaming de la médiathèque locale  
- **[Navidrome](./docker/NAS/navidrome.yml)** — Serveur de musique self-hosted  
- **[Servarr](./docker/NAS/servarr/)** — Gestion automatisée des séries et films  

<br>

### [Mac Mini](./docker/mac-mini/README.md) 

Dédié aux services consommateurs de ressources GPU/CPU intensives :

- **[Ollama + Open WebUI](./docker/mac-mini/open-webui.yml)** — Inférence LLM locale  
- **[Nextcloud + OnlyOffice](./docker/mac-mini/nextcloud.yml)** — Stockage et édition collaborative  
- **[Dozzle](./docker/mac-mini/dozzle.yml)** — Visualisation des logs Docker  
- **[Beszel Agent](./docker/mac-mini/beszel-agent.yml)** — Monitoring système  
- **[Diun](./docker/mac-mini/diun.yml)** — Alertes mises à jour Docker

<br>

### [VPS Cloud](./docker/vps-cloud/README.md)

Serveur cloud pour services à exposition publique.

- **[Portfolio](./docker/vps-cloud/portfolio.yml)** — Site vitrine  
- **[Umami](./docker/vps-cloud/umami.yml)** — Analytics self-hosted  

<br>

## Choix de conception

**Segmentation réseau stricte.** Chaque usage vit dans son VLAN avec une politique de forward explicite. Un compromis sur le réseau personnel n'a aucune visibilité sur le homelab, et vice versa. L'accès management est restreint à un branchement physique ou à une IP spécifique.

**Zéro port ouvert sur le WAN.** Tout l'accès distant passe par Tailscale. Cela élimine la surface d'attaque liée aux services exposés publiquement et simplifie drastiquement la gestion des certificats et des reverse proxies pour l'usage interne.

**Résilience des données en couches.** Les données critiques sont répliquées : code en local + cloud, fichiers sur NAS avec sauvegarde, VMs sauvegardées régulièrement. Aucun point de défaillance unique pour les données importantes.

**Infrastructure as Code / reproductibilité.** Les configurations sont versionnées (Docker Compose, scripts de déploiement) pour permettre une reconstruction rapide en cas de défaillance matérielle.

**Matériel low-power.** Le choix de mini PCs et de configurations allégées est délibéré : optimiser la consommation électrique pour une infrastructure qui tourne 24/7.
