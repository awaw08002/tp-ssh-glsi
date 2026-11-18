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

<img width="929" height="158" alt="Screenshot 2026-11-18 005613" src="https://github.com/user-attachments/assets/896a712e-b9ee-41f6-ab2d-f145cd935d0b" />



Vérification du service (état `active (running)`) :

<img width="928" height="343" alt="Screenshot 2026-11-18 005628" src="https://github.com/user-attachments/assets/60d6fc97-27a3-45f5-bdfe-f8d046a58e93" />



Vérification du port 22 :

<img width="926" height="91" alt="Screenshot 2026-11-18 005643" src="https://github.com/user-attachments/assets/e05932b0-eb62-4c42-a52d-79544333794d" />


**Explication** : le serveur OpenSSH est installé et actif. Il écoute sur le port 22 (état `LISTEN`).

---

## Étape 2 : Connexion depuis le client

Adresses IP du serveur :

<img width="368" height="33" alt="Screenshot 2026-11-18 005828" src="https://github.com/user-attachments/assets/d95d774e-74b7-412e-9788-db7869fcd247" />


Statut Tailscale :

<img width="448" height="67" alt="Screenshot 2026-11-18 005837" src="https://github.com/user-attachments/assets/0cf6d506-7099-4ad8-9351-2b76650c0b8c" />


Test de communication avec le client (ping) :

<img width="481" height="237" alt="Screenshot 2026-11-18 114128" src="https://github.com/user-attachments/assets/b16fad75-1b29-4bea-8e0c-48a6663be0eb" />


**Explication** : les deux machines n'étant pas sur le même réseau, nous avons utilisé Tailscale. Le ping répond sans perte de paquets, donc la liaison fonctionne. Le client se connecte ensuite avec `ssh awa@100.74.190.125`.

---

## Étape 3 : Configuration des clés SSH

Génération de la paire de clés sur le serveur :

    ssh-keygen -t rsa

<img width="932" height="410" alt="Screenshot 2026-11-18 113209" src="https://github.com/user-attachments/assets/c22fa596-c2bb-4d6e-b568-c811d8b53356" />


Vérification des fichiers créés (`id_rsa` : clé privée, `id_rsa.pub` : clé publique) :

<img width="924" height="123" alt="Screenshot 2026-11-18 113232" src="https://github.com/user-attachments/assets/a9d2f123-cfab-42f8-9992-cf060c336221" />


Envoi de la clé publique au client avec scp :

    scp id_rsa.pub ganadiop@100.112.180.28:~/

<img width="923" height="67" alt="Screenshot 2026-11-18 141843(1)" src="https://github.com/user-attachments/assets/73a8a856-fe5b-44ea-aa0e-0165c5053d1d" />


Test de connexion sans mot de passe :

<img width="422" height="44" alt="Screenshot 2026-11-18 182330" src="https://github.com/user-attachments/assets/328c34d4-a9ad-4077-933f-5db279aa7cba" />


**Explication** : la clé privée reste sur le serveur. Seule la clé publique est envoyée. Pour que la connexion ne demande plus de mot de passe, la clé publique doit être ajoutée au fichier `~/.ssh/authorized_keys` de la machine qui accepte la connexion.

---

## Étape 4 : Tunnel SSH

### Test avec le serveur web Apache

    sudo apt install apache2
    systemctl status apache2

<img width="920" height="401" alt="Screenshot 2026-11-18 010851" src="https://github.com/user-attachments/assets/e1b514ad-5587-4c9c-ad87-15c8ce59fbd6" />

<img width="929" height="387" alt="Screenshot 2026-11-18 183011" src="https://github.com/user-attachments/assets/1e21c81c-5b91-4cb9-9cae-7a9ff65b998b" />
<img width="931" height="410" alt="Screenshot 2026-11-18 183022" src="https://github.com/user-attachments/assets/95f03a60-1498-497a-8f99-8b1410a9b5ac" />



Le client crée le tunnel depuis son poste :

    ssh -L 1024:127.0.0.1:80 awa@100.74.190.125

puis ouvre `http://localhost:1024` dans son navigateur (voir la partie client).

### Test avec la base de données MariaDB

    sudo apt install mariadb-server
    systemctl status mariadb

<img width="928" height="413" alt="Screenshot 2026-11-18 152028" src="https://github.com/user-attachments/assets/43b9b7c1-e6af-45eb-9c12-e83bd836a3dd" />


Création de l'utilisateur de test (`tpuser`) :

<img width="926" height="247" alt="Screenshot 2026-11-18 171104" src="https://github.com/user-attachments/assets/62260618-f05a-4430-b61d-566d9972d007" />


Le client crée le tunnel :

    ssh -L 3307:127.0.0.1:3306 awa@100.74.190.125

puis se connecte avec :

    mysql -h 127.0.0.1 -P 3307 -u tpuser -p

Connexion du client vue depuis le serveur :

<img width="816" height="118" alt="Screenshot 2026-11-18 185316" src="https://github.com/user-attachments/assets/66bc2d07-bc6e-4fca-bff6-06c534c3c32a" />


**Explication** : un tunnel SSH fait passer d'autres protocoles (HTTP, MySQL) à l'intérieur d'une connexion SSH chiffrée. Apache (port 80) et MariaDB (port 3306) n'écoutent qu'en local sur le serveur, mais le client y accède à travers le tunnel, uniquement via le port 22.

---

## Conclusion

SSH permet d'accéder à une machine distante de façon sécurisée : par mot de passe ou par clé, et il peut aussi chiffrer d'autres services grâce aux tunnels.
