# Fiche technique N°2 – Installation et configuration de PHP

| Champ | Contenu |
|-------|---------|
| Tâche | Installation et configuration de PHP sur IIS |
| Date | 01/04/2026 |
| Responsable | Abraham |
| VM | server-web-dmz (112) – IP 10.0.11.22 |
| OS | Windows Server 2019 |

## 1. Objectif
Permettre au serveur web IIS d'exécuter des scripts PHP, nécessaires au bon fonctionnement du site de la Clinique Pasteur (architecture MVC).

## 2. Prérequis
- Serveur web IIS installé et fonctionnel (Fiche N°1)
- VM dans le réseau DMZ (10.0.11.22)
- Accès administrateur sur la VM

## 3. Procédure détaillée

### 3.1 – Téléchargement de PHP
1. Ouvrir Firefox sur la VM
2. Aller sur : https://windows.php.net/download/
3. Télécharger **PHP 8.5 (x64) – Non Thread Safe** (version ZIP)

> Pourquoi Non Thread Safe ? IIS utilise FastCGI, qui fonctionne mieux avec la version NTS.

### 3.2 – Installation de PHP
1. Créer le dossier `C:\PHP`
2. Extraire le contenu du ZIP dans `C:\PHP`
3. Vérifier que `C:\PHP\php-cgi.exe` existe

```cmd
dir C:\PHP\php-cgi.exe
```

### 3.3 – Installation du Visual C++ Redistributable
Télécharger et installer `vc_redist.x64.exe` depuis :
`aka.ms/vs/17/release/vc_redist.x64.exe`

> Sans ce runtime, PHP plante avec l'erreur 0xc0000135.

### 3.4 – Création du fichier php.ini
1. Dans `C:\PHP`, copier `php.ini-development` et renommer en `php.ini`
2. Ouvrir `php.ini` avec le Bloc-notes
3. Modifier :

```ini
extension_dir = "C:\PHP\ext"
```

4. Décommenter ces extensions :

```ini
extension=curl
extension=mbstring
extension=mysqli
extension=pdo_mysql
```

5. Ajouter le fuseau horaire :

```ini
date.timezone = Europe/Paris
```

### 3.5 – Installation du module CGI sur IIS

```powershell
Install-WindowsFeature -Name Web-CGI
iisreset
```

### 3.6 – Ajout du mappage PHP dans IIS
1. Ouvrir Gestionnaire IIS (`inetmgr`)
2. Double-cliquer sur **Mappage de gestionnaire**
3. Cliquer sur **Ajouter un mappage de module**

| Champ | Valeur |
|-------|--------|
| Chemin d'accès | `*.php` |
| Module | `FastCgiModule` |
| Exécutable | `C:\PHP\php-cgi.exe` |
| Nom | `PHP_via_FastCGI` |

### 3.7 – Document par défaut
1. IIS → **Document par défaut** → Ajouter `index.php`
2. Remonter `index.php` tout en haut

### 3.8 – Redémarrage IIS

```cmd
iisreset
```

## 4. Vérifications / Tests

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| PHP installé | `dir C:\PHP\php-cgi.exe` | Fichier présent |
| Module FastCGI actif | IIS → Modules | FastCgiModule dans la liste |
| Test PHP | Firefox → `https://localhost/info.php` | Page phpinfo() |

## 5. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| FastCgiModule non trouvé | `Install-WindowsFeature -Name Web-CGI` |
| Erreur 0xc0000135 | Installer `vc_redist.x64.exe` |
| Extensions PHP non chargées | Vérifier `extension_dir = "C:\PHP\ext"` |
| `Loaded Configuration File: (none)` | `setx /M PHPRC "C:\PHP"` puis `iisreset` |

## 6. Maintenance

| Action | Fréquence | Commande |
|--------|-----------|----------|
| Vérifier PHP | Hebdomadaire | `https://localhost/info.php` |
| Mettre à jour PHP | 6 mois / 1 an | Remplacer `C:\PHP` |
| Logs PHP | Mensuel | `C:\Windows\temp\php-errors.log` |

## 7. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| PHP 8.5 NTS | Recommandé pour IIS avec FastCGI |
| FastCGI | Module standard IIS pour exécuter PHP |
| Version x64 | Compatible Windows Server 2019 64 bits |
| mysqli / pdo_mysql | Connexion à la base de données MySQL |

## 8. Captures à ajouter
- Dossier `C:\PHP` avec `php-cgi.exe`
- Fichier `php.ini` modifié
- Mappage de gestionnaire avec `PHP_via_FastCGI`
- Page `https://localhost/info.php`
