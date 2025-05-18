# Migration de ZeroTier vers Tailscale

## ✨ Objectif

Remplacer l'ensemble du réseau ZeroTier par Tailscale, avec routage sécurisé via un subnet router configuré sur le serveur PVE (Proxmox).

---

## 🌐 Infrastructure

### Routeur Tailscale principal :

* **Nom** : `pve`
* **IP locale** : `192.168.1.201`
* **Rôle** : Subnet router
* **Commandes lancées** :

```bash
tailscale up --advertise-routes=192.168.1.0/24,192.168.192.0/24
```

* **IP Tailscale** : 100.xxx.xxx.xxx

### Sous-réseaux routés :

* `192.168.1.0/24` : Réseau principal (Docker, Proxmox, VMs)
* `192.168.192.0/24` : Réseau ZeroTier existant

---

## 🛠️ Procédure d'installation Tailscale (Linux)

> ⚠️ **Remarques importantes** :
>
> * Dans certains cas, `tailscaled` ne démarre pas automatiquement (surtout en LXC non privilégié) : il faut alors le lancer manuellement avec `--tun=userspace-networking`
> * Il peut être nécessaire de supprimer manuellement le socket `/var/run/tailscale/tailscaled.sock` si un processus précédent a échoué
> * Un port SSH personnalisé (ex. non standard sur MikroTik) peut bloquer le transfert `scp` si mal configuré
> * Certains fichiers `.npk` (MikroTik) ne sont pas compatibles selon l'architecture (`arm` vs `arm64`)

Sur chaque machine Debian/Ubuntu/DietPi/Docker-based :

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### Pour LXC non privilégié :

```bash
tailscaled --tun=userspace-networking --socks5-server=localhost:1055 &
tailscale up
```

### Pour LXC privilégié ou VM complète :

```bash
systemctl enable --now tailscaled
tailscale up
```

Une fois connecté, l’authentification se fait via le lien fourni (`https://login.tailscale.com/auth?...`).

---

## 🚀 Machines migrées vers Tailscale

| Machine            | Type       | IP Tailscale    | Accès SSH | Tailscale Installé  |
| ------------------ | ---------- | --------------- | --------- | ------------------- |
| `macbookpro`       | Mac M1     | 100.xxx.xxx.xxx | Oui       | Oui                 |
| `pve`              | Proxmox    | 100.xxx.xxx.xxx | Oui       | Oui (subnet router) |
| `omv`              | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui                 |
| `pbs`              | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui                 |
| `jellyfin`         | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui (userspace)     |
| `plex`             | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui (userspace)     |
| `docker-ptr`       | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui (userspace)     |
| `gcm`              | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui (userspace)     |
| `zmd`              | LXC Debian | 100.xxx.xxx.xxx | Oui       | Oui                 |
| `samsung-sm-f731b` | Mobile     | 100.xxx.xxx.xxx | -         | Oui (app Android)   |
| `win11`            | Windows    | 100.xxx.xxx.xxx | Oui       | Oui                 |

---

## 💔 ZeroTier : machines désactivées

| Machine      | Action effectuée   |
| ------------ | ------------------ |
| `macbookpro` | ZT supprimé        |
| `pve`        | ZT supprimé        |
| `omv`        | ZT supprimé        |
| `pbs`        | ZT supprimé        |
| `jellyfin`   | ZT supprimé        |
| `plex`       | ZT supprimé        |
| `docker-ptr` | ZT supprimé        |
| `gcm`        | ZT supprimé        |
| `zmd`        | ZT supprimé        |
| `zflip5`     | ZT désinstallé app |
| `win11`      | ZT supprimé        |

---

## 🚫 ZeroTier : machines restantes à nettoyer

**Aucune.**

> Le routeur MikroTik `MK` a été volontairement laissé de côté (abandonné).

---

## 🧪 Accès terminal via Tailscale

* Depuis le MacBook :

  ```bash
  ssh root@<ip_tailscale_machine>
  ```
* Exemple :

  ```bash
  ssh root@100.xxx.xxx.xxx  # plex
  ```

---

**La migration de l'ensemble de l'infrastructure de ZeroTier vers Tailscale a été réalisée avec succès : le réseau est maintenant plus sûr, centralisé, simple à maintenir et accessible depuis n'importe quel appareil autorisé.**
