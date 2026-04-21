# Documentation Clinique Pasteur — Projet 2

Infrastructure réseau sécurisée — BTS SIO SISR 2026  
**Lycée Gustave Flaubert | Abraham Emmanuel**

## Architecture réseau

| Réseau | Plage IP |
|--------|----------|
| LAN | 10.0.1.0/24 |
| DMZ | 10.0.11.0/24 |
| VPN | 10.0.20.0/24 |

## Machines virtuelles

| VM | Rôle | IP | OS |
|----|------|----|----|
| VM 112 | Serveur web (IIS) | 10.0.11.22 | Windows Server 2019 |
| VM 107 | Serveur interne (server-bdd) | 10.0.11.x | Debian 12 |
| VM | Supervision Nagios | 10.0.11.x | Debian 12 |

## Documentation

### Serveur web
- [Installation IIS](serveur-web/installation-iis.md)
- [Installation PHP + FastCGI](serveur-web/installation-php.md)
- [Déploiement du site](serveur-web/deploiement-site-web.md)

### Sécurité
- [Sécurisation HTTPS](securite/https-securisation.md)
- [Isolement DMZ](securite/isolement-dmz.md)
- [Contrôle des flux Stormshield](securite/firewall-controle-flux.md)
- [Sécurisation serveurs Debian](securite/securisation-serveurs-debian.md)

### Accès distant
- [VPN SSL Médecins](acces-distant/vpn-professionels.md)

### Supervision
- [Interface Nagios](supervision/interface-admin-supervision.md)
