# Fiche technique N°5 – Isolement du serveur web dans la DMZ

| Champ | Contenu |
|-------|---------|
| Tâche | Isolation du serveur web dans la zone DMZ |
| Date | 01/04/2026 |
| Responsable | Abraham (avec Mégane pour Stormshield) |
| VM | server-web-dmz (112) – IP 10.0.11.22 |
| Infrastructure | Proxmox + Stormshield |

## 1. Objectif
Isoler le serveur web dans une zone réseau dédiée (DMZ) afin qu'en cas de compromission, l'attaquant ne puisse pas accéder au réseau interne de la clinique.

## 2. Prérequis
- Serveur web installé (Fiches N°1 à N°3)
- Stormshield avec interface DMZ configurée
- Proxmox avec port réseau disponible

## 3. Architecture cible

```
[Internet]
│
▼
[Stormshield]
├── Port 2 (LAN)  → 10.0.1.254/24
└── Port 3 (DMZ)  → 10.0.11.254/24
         │
   [Proxmox 10.0.11.2]
         │
   [VM web 10.0.11.22]
```

| Flux | Autorisation | Justification |
|------|-------------|---------------|
| Internet → DMZ (443) | ✅ OUI | Accès patients au site |
| LAN → DMZ (443, 3389) | ✅ OUI | Administration techniciens |
| DMZ → LAN | ❌ NON | Isolation réseau interne |

## 4. Procédure détaillée

### 4.1 – Configuration IP Proxmox
```bash
# /etc/network/interfaces
auto vmbr0
iface vmbr0 inet static
    address 10.0.11.2/24
    gateway 10.0.11.254
    bridge-ports enp0s31f6
systemctl restart networking
```

### 4.2 – Configuration VM Windows
```cmd
netsh interface ip set address name="Ethernet" static 10.0.11.22 255.255.255.0 10.0.11.254
netsh interface ip set dns name="Ethernet" static 8.8.8.8
```

### 4.3 – Règles Stormshield (Mégane)

| Règle | Source | Destination | Service | Action |
|-------|--------|------------|---------|--------|
| 1 | LAN (10.0.1.0/24) | 10.0.11.22 | HTTPS, RDP | PASS |
| 2 | Internet | 10.0.11.22 | HTTPS | PASS |
| 3 | DMZ | LAN | ANY | BLOCK |

## 5. Vérifications / Tests

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| IP Proxmox | `ip a` | IP 10.0.11.2 |
| IP VM | `ipconfig` | IP 10.0.11.22 |
| Ping passerelle | `ping 10.0.11.254` | Réponse |
| Test isolation | `ping 10.0.1.254` depuis VM | ÉCHEC (BLOCK) |
| Accès HTTPS depuis LAN | `curl -k https://10.0.11.22` | Page du site |

## 6. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| Proxmox avec une seule interface | Ajout d'un câble entre Proxmox et Port 3 Stormshield |
| Interface DMZ désactivée | Activée par Mégane |
| Ping 8.8.8.8 ne fonctionnait pas | Règle DMZ → Internet manquante, ajoutée par Mégane |
| Perte accès interface web Proxmox | Reconfiguration via écran physique |

## 7. Maintenance

| Action | Fréquence | Méthode |
|--------|-----------|---------|
| Vérifier isolation DMZ | Mensuel | Ping 10.0.1.254 depuis VM (doit échouer) |
| Vérifier règles Stormshield | Mensuel | Interface admin Stormshield |
| Surveiller logs firewall | Hebdomadaire | Logs Stormshield |

## 8. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| DMZ sur port physique séparé | Isolation physique renforcée, aucun risque de fuite VLAN |
| BLOCK DMZ → LAN | Si le serveur web est piraté, l'attaquant ne peut pas accéder au LAN |
| IP fixe 10.0.11.22 | Règles firewall stables et prévisibles |

## 9. Captures à ajouter
- Stormshield – Interface DMZ activée
- Proxmox – `ip a` montrant 10.0.11.2
- VM Windows – `ipconfig` montrant 10.0.11.22
- Test ping 10.0.11.254 (réussi)
- Test ping 10.0.1.254 (échec – isolation OK)
- Règles de filtrage Stormshield
