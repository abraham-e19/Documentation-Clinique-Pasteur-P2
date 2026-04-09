# Fiche technique – Configuration des règles de filtrage firewall (Stormshield)

| Champ | Contenu |
|-------|---------|
| Tâche | Configuration des règles de filtrage du firewall |
| Date | 08/04/2026 |
| Responsable | Abraham |
| Matériel | Stormshield SN210W – IP 10.0.1.254 |
| Version | 4.8.4 |

## 1. Objectif
Mettre en place des règles de filtrage strictes sur le Stormshield afin de contrôler les flux entre Internet, la DMZ et le LAN, conformément au cahier des charges de la Clinique Pasteur. L'objectif est d'autoriser uniquement les communications nécessaires et de bloquer tout accès non autorisé.

## 2. Prérequis
- Accès à l'interface d'administration Stormshield (https://10.0.1.254)
- DMZ configurée sur le port 3 (10.0.11.254/24)
- Serveur web déployé en DMZ (SRV-WEB-DMZ – 10.0.11.22)

## 3. Architecture réseau

| Interface | Port | IP | Rôle |
|-----------|------|----|------|
| in | 2 | 10.0.1.254/24 | LAN (réseau interne) |
| out | 1 | 172.27.24.104/24 | Internet (WAN) |
| dmz1 | 3 | 10.0.11.254/24 | DMZ (serveur web) |

## 4. Matrice de flux

| Source | Destination | Service | Action | Justification |
|--------|------------|---------|--------|---------------|
| Network_internals (LAN) | SRV-WEB-DMZ | RDP, RDP_TCP | ✅ PASS | Administration du serveur web |
| Network_dmz1 (DMZ) | Internet | Any | ✅ PASS | Mises à jour serveur web |
| Internet | SRV-WEB-DMZ | HTTPS | ✅ PASS | Accès patients au site |
| Network_internals (LAN) | SRV-WEB-DMZ | HTTPS, SSH, Proxmox | ✅ PASS | Administration techniciens |
| Network_in (LAN) | SRV-MAIL-DMZ | HTTP, HTTPS | ✅ PASS | Accès serveur mail |
| SRV-WEB-DMZ | Network_internals (LAN) | Any | ❌ BLOCK | Isolation DMZ → LAN |

## 5. Procédure détaillée

### 5.1 – Accès à l'interface Stormshield
1. Ouvrir un navigateur depuis un poste LAN
2. Aller sur `https://10.0.1.254`
3. Se connecter avec les identifiants administrateur

### 5.2 – Accès aux règles de filtrage
1. Menu gauche → **Security Policy**
2. Onglet **Filtering**
3. Sélectionner la politique **ILOT1**

### 5.3 – Activation de la règle d'isolation DMZ → LAN
La règle 13 (**BLOCK SRV-WEB-DMZ → Network_internals**) était désactivée.
1. Localiser la règle 13
2. Cliquer sur le switch pour l'activer (OFF → ON)
3. Vérifier que l'action est bien **BLOCK**
4. Cliquer sur **APPLY** pour appliquer

## 6. Règles actives après configuration

| N° | Status | Action | Source | Destination | Port |
|----|--------|--------|--------|------------|------|
| 4 | ON | PASS | Network_internals | SRV-WEB-DMZ | RDP |
| 5 | ON | PASS | Network_dmz1 | Internet | Any |
| 6 | ON | PASS | Internet | SRV-WEB-DMZ | HTTPS |
| 7 | ON | PASS | Network_internals | SRV-WEB-DMZ | HTTPS, SSH, Proxmox |
| 8 | ON | PASS | Network_in | SRV-MAIL-DMZ | HTTP, HTTPS |
| 13 | ON | **BLOCK** | SRV-WEB-DMZ | Network_internals | Any |

## 7. Vérifications / Tests

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| Accès HTTPS depuis LAN | `curl -k https://10.0.11.22` depuis poste LAN | Page du site retournée |
| Isolation DMZ → LAN | `ping 10.0.1.254` depuis VM DMZ | Échec (BLOCK) |
| Accès Internet depuis DMZ | `ping 8.8.8.8` depuis VM DMZ | Réponse (PASS) |
| RDP depuis LAN vers DMZ | Connexion RDP vers 10.0.11.22 | Connexion établie |

## 8. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| Règle 16 activée en PASS (DMZ → LAN) | Désactivée car c'était l'inverse de ce qu'on voulait |
| Règle 13 BLOCK était désactivée | Activée pour assurer l'isolation DMZ → LAN |

## 9. Maintenance

| Action | Fréquence | Méthode |
|--------|-----------|---------|
| Vérifier les logs de filtrage | Hebdomadaire | Stormshield → Audit Logs |
| Contrôler isolation DMZ → LAN | Mensuel | Ping depuis VM DMZ vers LAN |
| Auditer les règles | Trimestriel | Vérifier qu'aucune règle non autorisée n'a été ajoutée |

## 10. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| BLOCK SRV-WEB-DMZ → LAN | Si le serveur web est compromis, l'attaquant ne peut pas rebondir vers le LAN interne |
| Seul HTTPS depuis Internet | Réduire la surface d'attaque, seul le port 443 est nécessaire |
| RDP autorisé LAN → DMZ | Permet la maintenance sans exposer RDP à Internet |
| SSH autorisé LAN → DMZ | Administration sécurisée du serveur web depuis le LAN |

## 11. Captures à ajouter
- Interface Stormshield – liste des règles actives
- Règle 13 BLOCK SRV-WEB-DMZ → Network_internals activée
- Test ping 10.0.1.254 depuis DMZ (échec – isolation OK)
- Test curl HTTPS depuis LAN (succès)
