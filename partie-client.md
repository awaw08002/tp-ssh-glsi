# Partie client (macOS)

**Auteur :** Gana Diop – MacBook Air, macOS
**Serveur du binôme (Awa) :** Ubuntu 26.04.1 LTS – adresse Tailscale `100.74.190.125`

---

## 1. Réseau privé avec Tailscale

Client et serveur ne sont pas dans le même lieu : les adresses locales (192.168.x.x) ne sont pas joignables entre elles. Nous utilisons **Tailscale** (VPN gratuit), qui donne à chaque machine une adresse privée `100.x.y.z`. Mon Mac a rejoint le réseau Tailscale de ma binôme.

L'application graphique n'a pas démarré sur mon Mac ; j'ai donc installé la version en ligne de commande :

```bash
brew install tailscale
sudo brew services start tailscale
sudo tailscale up
```

Vérification :

```bash
tailscale status
tailscale ip -4
```

![Tailscale](captures/01_tailscale.png)

---

## 2. Connexion SSH au serveur

```bash
ssh awa@100.74.190.125
```

1. À la première connexion, SSH affiche l'empreinte du serveur ; je réponds `yes`. Elle est enregistrée dans `~/.ssh/known_hosts`.
2. Je saisis le mot de passe du compte `awa`.
3. Le prompt devient `awa@awa:~$` : je suis connecté au serveur.

Vérification :

```bash
hostname
whoami
ip a
```

![Connexion SSH](captures/05_connexion_ssh.png)
![hostname, whoami, ip a](captures/05b_hostname_whoami.png)

---

## 3. Clés SSH : réception de la clé publique du serveur

Conformément au sujet, la paire de clés est **générée sur le serveur** (par ma binôme), puis sa **clé publique est envoyée sur ma machine** avec `scp`.

### 3.1 Préparer mon Mac pour recevoir le fichier

`scp` se connecte à mon Mac en SSH : il faut donc activer le serveur SSH de macOS.

**Réglages Système > Général > Partage > Session à distance** (activer).

Je donne ensuite à ma binôme mon nom d'utilisateur (`ganadiop`) et mon adresse Tailscale :

```bash
tailscale ip -4
```

Adresse de mon Mac : _à compléter_

![Session à distance activée](captures/06_session_distance.png)

### 3.2 Commande exécutée par ma binôme (sur le serveur)

```bash
scp ~/.ssh/id_ed25519.pub ganadiop@IP_TAILSCALE_DE_MON_MAC:/Users/ganadiop/
```

### 3.3 Vérification chez moi

```bash
ls -l ~/id_ed25519.pub
cat ~/id_ed25519.pub
```

![Clé publique reçue](captures/07_cle_recue.png)

---

## 4. Tunnel SSH

Principe : un tunnel SSH fait passer un autre protocole (HTTP, MySQL...) à l'intérieur de la connexion SSH chiffrée. Le port local `1024` de mon Mac est relié au port `80` du serveur.

### 4.1 Tunnel web (Apache)

```bash
ssh -L 1024:127.0.0.1:80 awa@100.74.190.125
```

Puis, dans le navigateur : `http://localhost:1024`.

![Tunnel web](captures/08_tunnel_web.png)
![Page Apache](captures/09_page_apache.png)

### 4.2 Tunnel base de données (MariaDB)

Terminal 1 :

```bash
ssh -L 3307:127.0.0.1:3306 awa@100.74.190.125
```

Terminal 2 :

```bash
mysql -h 127.0.0.1 -P 3307 -u tpuser -p tpdb
```

![Tunnel MariaDB](captures/10_tunnel_mariadb.png)
![Requêtes SQL](captures/11_requetes_sql.png)

---

## Annexe : paire de clés générée sur le client

En complément, j'ai aussi généré une paire de clés sur mon Mac. Dans ce sens (clé privée sur le client, clé publique copiée dans `~/.ssh/authorized_keys` du serveur avec `ssh-copy-id`), la connexion se fait sans mot de passe.

```bash
ssh-keygen -t ed25519
ls -a ~/.ssh
cat ~/.ssh/id_ed25519.pub
```

Empreinte :

```
SHA256:zJ2KKvS/7h4imbEH+FOqdv216lqa7RPD2A0L0wJK+zk ganadiop@macbookair.home
```

![ssh-keygen](captures/02_ssh-keygen.png)
![Contenu de ~/.ssh](captures/03_ls_ssh.png)
![Clé publique](captures/04_cle_publique.png)

> La clé privée n'est jamais partagée ni envoyée sur GitHub.

---

## Difficultés rencontrées

| Problème | Solution |
|---|---|
| Client et serveur dans des lieux différents | VPN Tailscale |
| L'application Tailscale ne démarrait pas | Installation en ligne de commande avec Homebrew |
| Mon Mac sur son propre réseau Tailscale | Rejoindre le réseau de ma binôme (`tailscale login`, puis approbation de son côté) |
| Mot de passe du compte `awa` refusé | Mot de passe redéfini côté serveur |
| MySQL déjà installé, conflit avec MariaDB | Utilisation du client `mysql` déjà présent |
