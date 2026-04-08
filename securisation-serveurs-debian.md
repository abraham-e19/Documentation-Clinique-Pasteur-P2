# Fiche technique – Sécurisation du serveur Debian (server-bdd)

| Champ | Contenu |
|-------|---------|
| Tâche | Sécurisation du serveur Debian interne |
| Date | 08/04/2026 |
| Responsable | Abraham |
| VM | server-bdd (107) |
| OS | Debian Bookworm |

## 1. Objectif
Sécuriser le serveur Debian hébergeant la base de données de la Clinique Pasteur afin de protéger les données de santé des patients contre les accès non autorisés.

## 2. Prérequis
- VM Debian installée sur Proxmox
- Accès root sur la VM
- Connexion Internet disponible

## 3. Procédure détaillée

### 3.1 – Mise à jour du système
```bash
apt update && apt upgrade -y
```
Met à jour tous les paquets installés pour corriger les failles de sécurité connues. 7 paquets mis à jour dont openssl et bind9.

### 3.2 – Désactivation d'Apache2
```bash
systemctl stop apache2
systemctl disable apache2
```
Apache2 était installé mais inutile sur ce serveur. On le désactive pour réduire la surface d'attaque.

### 3.3 – Configuration du pare-feu UFW
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
```
- `deny incoming` → bloque tout le trafic entrant par défaut
- `allow outgoing` → autorise tout le trafic sortant
- `allow ssh` → ouvre uniquement le port 22 pour l'administration
- `enable` → active le pare-feu au démarrage

### 3.4 – Sécurisation SSH
```bash
nano /etc/ssh/sshd_config
```
Modification de la ligne :
```
PermitRootLogin no
```
Empêche toute connexion SSH directe en root. Même si le mot de passe root est compromis, l'attaquant ne peut pas se connecter directement.
```bash
systemctl restart ssh
```
Redémarre SSH pour appliquer la nouvelle configuration.

### 3.5 – Installation de Fail2ban
```bash
apt install fail2ban -y
```
Fail2ban surveille les logs SSH et bloque automatiquement les IPs qui font trop de tentatives de connexion échouées (protection contre les attaques brute force).

## 4. Vérifications / Tests

| Test | Commande | Résultat attendu |
|------|----------|-----------------|
| UFW actif | `ufw status` | Status: active, port 22 ALLOW |
| Apache2 désactivé | `systemctl status apache2` | inactive (dead) |
| Fail2ban actif | `systemctl status fail2ban` | active (running) |
| SSH sécurisé | `grep PermitRootLogin /etc/ssh/sshd_config` | PermitRootLogin no |

## 5. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| Faute de frappe `upgarde` | Corriger en `upgrade` |

## 6. Maintenance

| Action | Fréquence | Commande |
|--------|-----------|----------|
| Vérifier les logs Fail2ban | Hebdomadaire | `fail2ban-client status sshd` |
| Vérifier les règles UFW | Mensuel | `ufw status` |
| Mettre à jour le système | Mensuel | `apt update && apt upgrade -y` |

## 7. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| UFW | Pare-feu simple à configurer sur Debian, idéal pour bloquer les ports inutiles |
| PermitRootLogin no | Empêche les connexions SSH directes en root, réduit le risque d'intrusion |
| Fail2ban | Protection automatique contre les attaques brute force sur SSH |
| Désactivation Apache2 | Réduire la surface d'attaque en supprimant les services inutiles |

## 8. Captures à ajouter
- `ufw status` → pare-feu actif avec port 22
- `systemctl status fail2ban` → service actif
- `systemctl status apache2` → service désactivé
- Fichier `sshd_config` avec `PermitRootLogin no`
