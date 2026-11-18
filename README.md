# TP SSH – Mise en place du protocole SSH

**DGI-ESP / UCAD – L3 GLSI – Services Réseaux Avancés – 2026-2027**
Enseignant : Dr. GUEYE

**Objectif :** comprendre comment utiliser SSH pour sécuriser l'accès à une machine distante : connexion, clés SSH et tunnel.

---

## Binôme

| Rôle | Étudiant(e) | Système | Adresse Tailscale |
|---|---|---|---|
| **Serveur SSH** | Awa War | Ubuntu 26.04.1 LTS | `100.74.190.125` |
| **Client SSH** | Gana Diop | macOS | `100.112.180.28` |

---

## Contenu du dépôt

| Fichier | Contenu |
|---|---|
| [`partie-serveur.md`](partie-serveur.md) | Awa War : installation de SSH, génération des clés, envoi de la clé publique, Apache et MariaDB |
| [`partie-client.md`](partie-client.md) | Gana Diop : connexion SSH, réception de la clé publique, tunnels web et base de données |
| [`captures/`](captures/) | Captures d'écran des deux parties |

---

## Architecture

Le client et le serveur ne sont pas dans le même lieu. Les adresses locales (192.168.x.x) du sujet ne sont donc pas joignables entre les deux machines. Elles sont remplacées par des adresses **Tailscale**, un VPN gratuit qui donne à chaque machine une adresse privée de la forme `100.x.y.z`.

```
   Client (macOS)                                     Serveur (Ubuntu)
   Gana Diop                                          Awa War
   ssh, navigateur, client MySQL                      openssh-server, Apache, MariaDB

        |                                                   |
        +=============== réseau privé Tailscale =============+
                      connexion SSH chiffrée (port 22)
```

---

## Déroulement du TP

### Étape 1 : installation de SSH (serveur)

Installation et démarrage du serveur `openssh-server`, vérification du service et de l'ouverture du port 22.
Détails dans [`partie-serveur.md`](partie-serveur.md).

### Étape 2 : connexion SSH depuis le client

Connexion du client macOS au serveur avec `ssh utilisateur@adresse_du_serveur`.
Détails dans [`partie-client.md`](partie-client.md).

### Étape 3 : configuration des clés SSH

Conformément au sujet, la paire de clés est générée **sur le serveur** (`ssh-keygen`), puis la clé publique est envoyée au répertoire personnel du client avec `scp`.
Détails dans [`partie-serveur.md`](partie-serveur.md) et [`partie-client.md`](partie-client.md).

### Étape 4 : tunnel SSH

Un tunnel SSH fait passer un autre protocole (HTTP, MySQL...) à l'intérieur de la connexion SSH chiffrée.

| Tunnel | Commande (côté client) | Test |
|---|---|---|
| Web (Apache) | `ssh -L 1024:127.0.0.1:80 utilisateur@IP_SERVEUR` | `http://localhost:1024` dans le navigateur |
| Base de données (MariaDB) | `ssh -L 3307:127.0.0.1:3306 utilisateur@IP_SERVEUR` | `mysql -h 127.0.0.1 -P 3307 -u tpuser -p tpdb` |

Détails dans [`partie-client.md`](partie-client.md).

---

## Avancement

| Étape | Responsable | État |
|---|---|---|
| Réseau privé Tailscale | Les deux | Fait |
| 1. Installation de SSH | Awa War | _à compléter_ |
| 2. Connexion SSH du client au serveur | Gana Diop | Fait |
| 3. Génération des clés sur le serveur | Awa War | À faire |
| 3. Envoi et réception de la clé publique | Awa War / Gana Diop | À faire |
| 4. Tunnel web | Gana Diop | À faire |
| 4. Tunnel base de données | Gana Diop | À faire |

---

## Remarque sur les adresses IP

Les captures du sujet utilisent des adresses 192.168.1.x. Dans notre cas, les deux machines sont à distance : toutes les connexions passent par les adresses **Tailscale** (100.x.y.z).