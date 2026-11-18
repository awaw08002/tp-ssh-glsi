# Partie serveur

**IP Tailscale du serveur** : 100.74.190.125
**IP Tailscale du client** : 100.112.180.28

Le TP a été réalisé à distance : le serveur et le client sont chez des personnes différentes, reliés par un VPN Tailscale (adresses en 100.x.x.x).

---

## Étape 1 : Installation du serveur SSH

Commandes :

    sudo apt update
    sudo apt install openssh-server
    sudo systemctl enable ssh
    sudo systemctl start ssh

Vérification de la version installée :

<img width="925" height="439" alt="Screenshot 2026-11-18 004612" src="https://github.com/user-attachments/assets/b26bc7a9-dfe4-43f0-9c70-5aebb4aa8c0d" />


Vérification du service (état `active (running)`) :

![statut ssh](captures/serveur-02-ssh-status.png)

Vérification du port 22 :

![port 22](captures/serveur-03-port-22.png)

**Explication** : le serveur OpenSSH est installé et actif. Il écoute sur le port 22 (état `LISTEN`).

---

## Étape 2 : Connexion depuis le client

Adresses IP du serveur :

![adresses ip](captures/serveur-04-ip.png)

Statut Tailscale :

![tailscale](captures/serveur-05-tailscale-status.png)

Test de communication avec le client (ping) :

![ping client](captures/serveur-04b-ping-client.png)

**Explication** : les deux machines n'étant pas sur le même réseau, nous avons utilisé Tailscale. Le ping répond sans perte de paquets, donc la liaison fonctionne. Le client se connecte ensuite avec `ssh awa@100.74.190.125`.

---

## Étape 3 : Configuration des clés SSH

Génération de la paire de clés sur le serveur :

    ssh-keygen -t rsa

![keygen](captures/serveur-06-keygen.png)

Vérification des fichiers créés (`id_rsa` : clé privée, `id_rsa.pub` : clé publique) :

![ls ssh](captures/serveur-07-ls-ssh.png)

Envoi de la clé publique au client avec scp :

    scp id_rsa.pub ganadiop@100.112.180.28:~/

![scp](captures/serveur-08-scp.png)

Test de connexion sans mot de passe :

![sans mot de passe](captures/serveur-09-connexion-sans-mdp.png)

**Explication** : la clé privée reste sur le serveur. Seule la clé publique est envoyée. Pour que la connexion ne demande plus de mot de passe, la clé publique doit être ajoutée au fichier `~/.ssh/authorized_keys` de la machine qui accepte la connexion.

---

## Étape 4 : Tunnel SSH

### Test avec le serveur web Apache

    sudo apt install apache2
    systemctl status apache2

![apache](captures/serveur-10-apache.png)
![curl](captures/serveur-10b-curl.png)

Le client crée le tunnel depuis son poste :

    ssh -L 1024:127.0.0.1:80 awa@100.74.190.125

puis ouvre `http://localhost:1024` dans son navigateur (voir la partie client).

### Test avec la base de données MariaDB

    sudo apt install mariadb-server
    systemctl status mariadb

![mariadb](captures/serveur-11-mariadb.png)

Création de l'utilisateur de test (`tpuser`) :

![utilisateur](captures/serveur-12-mariadb-user.png)

Le client crée le tunnel :

    ssh -L 3307:127.0.0.1:3306 awa@100.74.190.125

puis se connecte avec :

    mysql -h 127.0.0.1 -P 3307 -u tpuser -p

Connexion du client vue depuis le serveur :

![processlist](captures/serveur-13-processlist.png)

**Explication** : un tunnel SSH fait passer d'autres protocoles (HTTP, MySQL) à l'intérieur d'une connexion SSH chiffrée. Apache (port 80) et MariaDB (port 3306) n'écoutent qu'en local sur le serveur, mais le client y accède à travers le tunnel, uniquement via le port 22.

---

## Conclusion

SSH permet d'accéder à une machine distante de façon sécurisée : par mot de passe ou par clé, et il peut aussi chiffrer d'autres services grâce aux tunnels.
