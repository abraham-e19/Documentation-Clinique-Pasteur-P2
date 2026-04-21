# Fiche technique N°4 – Mise en place du chiffrement HTTPS

| Champ | Contenu |
|-------|---------|
| Tâche | Mise en place du chiffrement HTTPS sur le serveur web |
| Date | 01/04/2026 |
| Responsable | Abraham |
| VM | server-web-dmz (112) – IP 10.0.11.22 |
| OS | Windows Server 2019 |

## 1. Objectif
Sécuriser les échanges entre les patients et le site web de la Clinique Pasteur en activant HTTPS (port 443) avec un certificat SSL/TLS.

## 2. Prérequis
- IIS installé (Fiche N°1)
- VM dans la DMZ (10.0.11.22)
- Accès administrateur

## 3. Procédure détaillée

### 3.1 – Création du certificat auto-signé
```powershell
New-SelfSignedCertificate -DnsName "serverweb-pasteur" -CertStoreLocation "Cert:\LocalMachine\My"
```
Noter le Thumbprint affiché.

### 3.2 – Ajout dans le magasin racine
```powershell
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Thumbprint -eq "VOTRE_THUMBPRINT"}
$store = New-Object System.Security.Cryptography.X509Certificates.X509Store("Root","LocalMachine")
$store.Open("ReadWrite")
$store.Add($cert)
$store.Close()
```

### 3.3 – Liaison du certificat au port 443
```cmd
netsh http add sslcert ipport=0.0.0.0:443 certhash=VOTRE_THUMBPRINT appid={00000000-0000-0000-0000-000000000000}
```

### 3.4 – Règle pare-feu
```cmd
netsh advfirewall firewall add rule name="HTTPS" dir=in action=allow protocol=TCP localport=443
```

### 3.5 – Configuration IIS
1. Gestionnaire IIS → Sites → Default Web Site → **Liaisons**
2. Ajouter → Type : `https` → Port : `443`
3. Certificat SSL : choisir `serverweb-pasteur` → OK

## 4. Vérifications / Tests

| Test | Commande | Résultat attendu |
|------|----------|-----------------|
| Port 443 ouvert | `netstat -an \| find ":443"` | LISTENING |
| Certificat installé | `Get-ChildItem -Path Cert:\LocalMachine\My` | Certificat affiché |
| HTTPS local | Firefox → `https://localhost` | Page avec cadenas |

## 5. Difficultés rencontrées

| Problème | Solution |
|----------|----------|
| `New-SelfSignedCertificate` non reconnue | Utiliser PowerShell au lieu de CMD |
| Erreur `ArgumentNullException` | Thumbprint incorrect → récupéré avec `Get-ChildItem` |
| Erreur 403.14 | `index.html` non reconnu comme document par défaut → ajouté dans IIS |

## 6. Maintenance

| Action | Fréquence | Commande |
|--------|-----------|----------|
| Vérifier expiration certificat | Mensuel | `Get-ChildItem -Path Cert:\LocalMachine\My` |
| Renouveler certificat | 1 an | Recréer et relier au port 443 |
| Vérifier port 443 | Hebdomadaire | `netstat -an \| find ":443"` |

## 7. Justifications des choix techniques

| Choix | Pourquoi |
|-------|----------|
| Certificat auto-signé | Phase de test/démonstration, valide le fonctionnement HTTPS |
| Port 443 | Port standard HTTPS, compatible tous navigateurs |
| Liaison 0.0.0.0:443 | Écoute sur toutes les interfaces réseau de la VM |

> Note pour l'oral : "En production, je remplacerai ce certificat auto-signé par Let's Encrypt pour qu'il soit reconnu automatiquement par les navigateurs."

## 8. Captures à ajouter
- PowerShell après création du certificat (avec Thumbprint)
- Commande `netsh http add sslcert` réussie
- Liaison HTTPS dans IIS
- Firefox sur `https://localhost` avec cadenas
