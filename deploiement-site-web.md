# Fiche technique N°3 – Déploiement du site web

| Champ | Contenu |
|-------|---------|
| Tâche | Déploiement du site PHP fourni par les développeurs |
| Date | 01/04/2026 |
| Responsable | Abraham |
| VM | server-web-dmz (112) – IP 10.0.11.22 |
| OS | Windows Server 2019 |

## 1. Objectif
Mettre en ligne le site web de la Clinique Pasteur développé par l'équipe SLAM, en remplacement de la page par défaut IIS.

## 2. Prérequis
- IIS installé (Fiche N°1)
- PHP configuré (Fiche N°2)
- Fichiers du site disponibles

## 3. Procédure détaillée

### 3.1 – Nettoyage du dossier wwwroot
```cmd
del C:\inetpub\wwwroot\iisstart.* /Q
del C:\inetpub\wwwroot\index.html /Q
```

### 3.2 – Copie des fichiers du site
```cmd
xcopy "C:\Users\Administrateur\Downloads\site\*" "C:\inetpub\wwwroot\" /E /I /Y
```

### 3.3 – Vérification de la copie
```cmd
dir C:\inetpub\wwwroot
```
Doit afficher : `controller/`, `model/`, `view/`, `public/`, `index.php`

### 3.4 – Permissions
```cmd
icacls C:\inetpub\wwwroot /grant "IUSR:(OI)(CI)R"
icacls C:\inetpub\wwwroot /grant "IIS_IUSRS:(OI)(CI)R"
```

### 3.5 – Redémarrage IIS
```cmd
iisreset
```

## 4. Vérifications / Tests

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| Fichiers présents | `dir C:\inetpub\wwwroot\*.php` | Affiche index.php |
| Page d'accueil | Firefox → `https://localhost` | Site avec les 3 boutons (Patient, Médecin, Assistante) |
| Ressources CSS | Inspecter → Network | Fichiers CSS/JS sans erreur 404 |

## 5. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| Erreur 403.14 | `index.php` absent des documents par défaut → ajouté dans IIS |
| Erreur 500.0 FastCGI (0xc0000135) | Visual C++ Redistributable manquant → installé |
| `could not find driver` | Extensions `pdo_mysql` et `mysqli` non activées dans `php.ini` |
| Site sans CSS | Fichiers CSS dans `public/css/` → chemin correct après copie complète |

## 6. Maintenance

| Action | Fréquence | Méthode |
|--------|-----------|---------|
| Mettre à jour le site | À chaque version | Remplacer fichiers dans `C:\inetpub\wwwroot` |
| Sauvegarder le site | Hebdomadaire | Copier `wwwroot` vers dossier backup |
| Logs d'erreur | Hebdomadaire | `C:\inetpub\logs\LogFiles` |

## 7. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| xcopy | Copie toute l'arborescence en une seule commande |
| Architecture MVC | Bonne pratique de développement (controller, model, view) |
| index.php document par défaut | PHP est le langage du site, point d'entrée = index.php |
| Permissions IUSR | L'utilisateur anonyme IIS doit pouvoir lire les fichiers |

## 8. Captures à ajouter
- Dossier wwwroot après copie
- Page d'accueil du site dans Firefox
- Onglet Network sans erreur 404
- Document par défaut avec index.php en première position
