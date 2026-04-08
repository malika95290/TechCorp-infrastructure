# 🏢 TechCorp : Infrastructure IT complète d'une PME

> Ce dépôt documente la conception, le déploiement et l'administration de l'infrastructure IT complète d'une PME fictive de 50 personnes, répartie sur deux sites (Paris et Lyon).

---

## 🎯 Objectifs du projet

Ce projet couvre l'ensemble des compétences d'un administrateur systèmes et réseaux :

- Conception d'une **topologie réseau** avec segmentation par VLANs et DMZ
- Administration de serveurs **Windows Server** (Active Directory, GPO, DNS, DHCP)
- Administration de serveurs **Linux** (Samba, SSH, Apache, Fail2Ban)
- Mise en place d'un **VPN** site-à-site et d'accès distant (WireGuard)
- Configuration d'un **pare-feu** avec règles de filtrage par zone (pfSense / iptables)
- **Scripting** Bash, PowerShell et Python pour automatiser l'exploitation
- Déploiement d'un **système de ticketing** (GLPI) et d'un **intranet**
- **Supervision** de l'infrastructure avec Zabbix

---

## 🗂️ Structure du dépôt

```
techcorp-infrastructure/
├── 01-reseau/              # Topologie Cisco Packet Tracer, plan d'adressage
├── 02-serveurs/            # Windows Server (AD, GPO) et Linux (Samba, SSH)
├── 03-vpn/                 # Configuration WireGuard site-à-site et accès distant
├── 04-firewall/            # Règles pfSense / iptables
├── 05-scripts/             # Scripts Bash, PowerShell et Python
├── 06-services/            # GLPI (ticketing) et intranet
├── 07-supervision/         # Zabbix — dashboards et alertes
└── docs/                   # Schémas, captures d'écran, documentation générale
```

---

## 🌐 Module 1 : Topologie réseau

**Outil :** Cisco Packet Tracer

### Architecture

```
         Internet
             |
        [Routeur WAN]
             |
        [Pare-feu]
          /      \
       [DMZ]    [Switch multicouche]
         |         |        |        |
      Web/Mail  VLAN10   VLAN20   VLAN30
               Direction  RH       IT
                                    |
                                 VLAN40
                                Serveurs

        Site Lyon (filiale)
             |
        [Routeur Lyon] <-- VPN site-à-site --> [Routeur Paris]
          /       \
       VLAN50   VLAN60
      Commercial Support
```

### Plan d'adressage

| VLAN | Nom | Réseau | Passerelle |
|------|-----|--------|------------|
| 10 | Direction | 192.168.10.0/24 | 192.168.10.1 |
| 20 | RH | 192.168.20.0/24 | 192.168.20.1 |
| 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| 40 | Serveurs | 192.168.40.0/24 | 192.168.40.1 |
| 50 | Commercial (Lyon) | 192.168.50.0/24 | 192.168.50.1 |
| 60 | Support (Lyon) | 192.168.60.0/24 | 192.168.60.1 |
| 100 | DMZ | 192.168.100.0/24 | 192.168.100.1 |
| WAN | Lien Internet | 203.0.113.0/30 | 203.0.113.1 |

### Règles de sécurité appliquées

- Le VLAN RH (20) ne peut pas communiquer avec le VLAN IT (30)
- Seul le VLAN IT (30) peut accéder au VLAN Serveurs (40)
- La DMZ est isolée du LAN interne (aucun flux retour autorisé)
- Le VPN chiffre tout le trafic inter-sites Paris / Lyon
- NAT sur le routeur WAN pour masquer les adresses internes

📁 [Voir le dossier 01-reseau](./01-reseau/)

---

## 🖥️ Module 2 : Serveurs

### Windows Server 2022 : Active Directory

| Élément | Configuration |
|---------|--------------|
| Domaine | techcorp.local |
| Unités d'Organisation | Direction, RH, IT, Commercial |
| GPO Direction | Fond d'écran imposé, verrouillage session 5 min |
| GPO RH | Restriction ports USB, politique mot de passe renforcée |
| GPO IT | Accès PowerShell autorisé, outils d'administration |
| DHCP | Scopes par VLAN avec réservations pour les serveurs |
| DNS | Zone directe et inverse pour techcorp.local |

### Ubuntu Server : Services réseau

| Service | Rôle |
|---------|------|
| Samba | Partage de fichiers cross-platform avec permissions par groupe |
| SSH | Accès distant sécurisé (clé uniquement, mot de passe désactivé) |
| Nginx | Hébergement de l'intranet |
| NTP | Synchronisation de l'heure sur tout le réseau |
| Fail2Ban | Blocage automatique après 3 tentatives SSH échouées |

📁 [Voir le dossier 02-serveurs](./02-serveurs/)

---

## 🔒 Module 3 : VPN WireGuard

Mise en place d'un VPN double usage :
- **Site-à-site** : tunnel permanent Paris / Lyon
- **Accès distant** : connexion sécurisée pour les télétravailleurs

```
Télétravailleur                  Site Paris
     |                               |
[Client WireGuard] <-- VPN --> [Serveur WireGuard]
     |                               |
  Internet                     LAN 192.168.x.x
```

📁 [Voir le dossier 03-vpn](./03-vpn/)

---

## 🛡️ Module 4 : Pare-feu pfSense

| Interface | Zone | Politique |
|-----------|------|-----------|
| WAN | Internet | DROP par défaut |
| LAN | Réseau interne | Autorisation contrôlée |
| DMZ | Serveurs exposés | HTTP/HTTPS uniquement depuis WAN |
| VPN | Accès distant | Authentification obligatoire |

Fonctionnalités configurées :
- Filtrage stateful par interface
- NAT sortant pour les VLANs internes
- Portail captif pour le Wi-Fi invité
- IDS/IPS avec Snort (détection d'intrusion)
- Logs centralisés des connexions refusées

📁 [Voir le dossier 04-firewall](./04-firewall/)

---

## ⚙️ Module 5 : Scripts

| Fichier | Langage | Description |
|---------|---------|-------------|
| `surveillance.sh` | Bash | Surveillance CPU, RAM, disque, services critiques avec alertes mail |
| `sauvegarde.sh` | Bash | Sauvegarde incrémentale Samba avec rsync, rotation 7 jours, rapport |
| `gestion-users.ps1` | PowerShell | Création automatique de comptes AD depuis un CSV |
| `scanner-reseau.py` | Python | Scan du réseau local, détection de nouvelles machines, export CSV/HTML |

📁 [Voir le dossier 05-scripts](./05-scripts/)

---

## 🎫 Module 6 : Services internes

### GLPI : Système de ticketing

- Installation sur Ubuntu Server avec base de données MariaDB
- Authentification via LDAP (Active Directory)
- Catégories : Réseau, Poste de travail, Logiciel, Accès, Sécurité
- SLA configurés : urgent (4h), normal (24h), faible (72h)
- Tableau de bord avec statistiques des tickets par catégorie et par technicien

### Intranet TechCorp

- Hébergé sur Nginx
- Pages : Annuaire employés, Procédures IT, Contacts, État des services
- Page de statut des services générée automatiquement par un script Python

📁 [Voir le dossier 06-services](./06-services/)

---

## 📊 Module 7 : Supervision Zabbix

Éléments supervisés :

| Hôte | Métriques surveillées |
|------|----------------------|
| Serveur Windows | CPU, RAM, disque, services AD/DNS/DHCP |
| Serveur Linux | CPU, RAM, disque, Samba, SSH, Nginx, Fail2Ban |
| Routeurs Cisco | Disponibilité, trafic interfaces, état VPN |
| Pare-feu pfSense | Connexions actives, règles déclenchées |

Alertes configurées :
- Hôte inaccessible depuis plus de 5 minutes
- Espace disque supérieur à 80 %
- Service critique arrêté
- Tentative d'intrusion détectée par Snort

📁 [Voir le dossier 07-supervision](./07-supervision/)

---

## 🛠️ Technologies utilisées

| Catégorie | Outils |
|-----------|--------|
| Réseau | Cisco Packet Tracer, pfSense, WireGuard |
| Systèmes | Windows Server 2022, Ubuntu Server 24.04 |
| Virtualisation | VirtualBox |
| Scripting | Bash, PowerShell, Python |
| Services | Active Directory, Samba, Nginx, GLPI, Zabbix |
| Sécurité | Fail2Ban, Snort, iptables |
| Monitoring | Zabbix, Grafana |

---

## 👩‍💻 Auteure

**Malika AMRANI**  

---

## 📅 Avancement

- [x] Module 1 — Topologie réseau Cisco
- [ ] Module 2 — Serveurs Windows et Linux
- [ ] Module 3 — VPN WireGuard
- [ ] Module 4 — Pare-feu pfSense
- [ ] Module 5 — Scripts Bash, PowerShell, Python
- [ ] Module 6 — GLPI et intranet
- [ ] Module 7 — Supervision Zabbix
