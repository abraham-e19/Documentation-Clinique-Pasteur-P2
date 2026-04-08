# Fiche technique N°1 – Installation du serveur web (IIS)

| Champ | Contenu |
|-------|---------|
| Tâche | Installation du serveur web IIS sur Windows Server |
| Date | 27/03/2026 |
| Responsable | Abraham |
| VM | server-web-dmz (112) – IP 10.0.11.22 |
| OS | Windows Server 2019 |

## 1. Objectif
Mettre en place un serveur web IIS capable d'héberger le site patients de la Clinique Pasteur, avec support PHP et HTTPS. Ce serveur est placé dans la DMZ afin d'isoler le service web du réseau interne.

## 2. Prérequis
- VM Windows Server 2019 installée sur Proxmox
- VM configurée dans le réseau DMZ (10.0.11.x)
- Accès administrateur sur la VM
- Connexion Internet pour les mises à jour

## 3. Procédure détaillée

### 3.1 – Connexion à la VM
Via la console Proxmox ou en RDP :
- Adresse : 10.0.11.22
- Identifiant : Administrateur

### 3.2 – Ouverture du Gestionnaire de serveur
Le Gestionnaire de serveur s'ouvre automatiquement au démarrage. Sinon, cliquer sur l'icône dans la barre des tâches.

### 3.3 – Ajout du rôle IIS
1. Cliquer sur **Ajouter des rôles et fonctionnalités**
2. Type d'installation → Installation basée sur un rôle ou une fonctionnalité
3. Sélection du serveur → Choisir le serveur local
4. Rôles du serveur → Cocher **Serveur Web (IIS)**
5. Fenêtre pop-up → Cliquer sur **Ajouter des fonctionnalités**
6. Confirmation → Cliquer sur **Installer**

### 3.4 – Vérification de l'installation
```cmd
net start w3svc
```
Résultat attendu : "Le service demandé a déjà été démarré."

### 3.5 – Test de la page par défaut
1. Ouvrir Firefox sur la VM
2. Taper : `http://localhost`
3. La page d'accueil IIS doit s'afficher

## 4. Vérifications / Tests

| Test | Commande | Résultat attendu |
|------|----------|-----------------|
| Service IIS actif | `net start w3svc` | Service déjà démarré |
| Accès HTTP local | `curl http://localhost` | Page HTML retournée |
| Dossier wwwroot existe | `dir C:\inetpub\wwwroot` | Dossier présent avec iisstart.htm |

## 5. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| Aucune – installation sans erreur | – |

## 6. Maintenance

| Action | Fréquence | Commande |
|--------|-----------|----------|
| Redémarrer IIS | En cas de problème | `iisreset` |
| Vérifier les logs d'erreur | Hebdomadaire | `C:\inetpub\logs\LogFiles` |
| Vérifier l'espace disque | Mensuel | `dir C:\inetpub` |

## 7. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| Windows Server 2019 | OS stable, support longue durée, compatible PHP et outils Microsoft |
| IIS | Serveur web intégré à Windows, facile à configurer, supporte PHP via FastCGI |
| VM dans DMZ | Isolation du serveur web pour protéger le réseau interne |
| IP fixe 10.0.11.22 | Configuration stable et prévisible pour le routage et les règles firewall |

## 8. Captures à ajouter
- Gestionnaire de serveur avant installation
- Sélection du rôle IIS
- Page `http://localhost` après installation
- Dossier `C:\inetpub\wwwroot`
