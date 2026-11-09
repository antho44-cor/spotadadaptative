# Ma musique — essai en local (Docker + Caddy)

Un conteneur Caddy sert le dossier `site/` en HTTPS sur ton réseau local, avec un certificat signé par sa propre autorité (« tls internal »). Il faut donc : un nom qui pointe vers la machine, puis faire confiance à l'autorité sur l'iPad.

Ports par défaut, peu courants : **8743** (HTTPS) et **8780** (HTTP, uniquement pour télécharger le certificat). Ils se changent avec `HTTPS_PORT` et `HTTP_PORT`.

> Cette configuration n'a pas pu être exécutée là où elle a été écrite (pas de démon Docker). Si Caddy ne démarre pas : `docker compose logs`.

## 1. Un nom pour la machine

L'iPad doit résoudre `musique.lan` vers l'IP de la machine qui fait tourner Docker.

- **OpenWrt / dnsmasq** : ajouter une entrée d'hôte `musique.lan` → IP de la machine (Réseau › DHCP et DNS › Entrées d'hôtes). Pi-hole ou AdGuard : « DNS local » de la même façon.
- **Sans toucher au DNS** : utiliser le nom mDNS de la machine (Mac : natif ; Linux : avahi), par exemple `SITE_HOST=mon-serveur.local`.
- **Dernier recours** : l'IP elle-même dans `SITE_HOST`. Je ne sais pas si le tableau de bord Spotify accepte une adresse `https://` avec une IP, donc à tester en dernier.

## 2. Démarrer

```
docker compose up -d
```

Autres valeurs : `HTTPS_PORT=9555 HTTP_PORT=9556 SITE_HOST=mon-serveur.local docker compose up -d`. Pense à ouvrir ces ports dans le pare-feu de la machine si elle en a un.

## 3. Faire confiance au certificat sur l'iPad

1. Safari → `http://musique.lan:8780/root.crt` → **Autoriser** le téléchargement du profil.
2. **Réglages › Profil téléchargé › Installer**.
3. **Réglages › Général › Informations › Réglages des certificats de confiance** → activer le certificat de l'autorité Caddy.

Si le téléchargement ne se propose pas : `docker compose cp caddy:/data/caddy/pki/authorities/local/root.crt ./root.crt`, puis envoyer ce fichier sur l'iPad (AirDrop, e-mail).

## 4. Ouvrir l'application

`https://musique.lan:8743/` dans Safari : le cadenas doit être normal. Sans Client ID, l'application tourne en **mode démo** : c'est le bon moment pour juger la taille des tuiles et le tactile.

## 5. Brancher Spotify

- Dans Réglages de l'application (maintenir le bouton 2 s), copier l'**adresse de redirection** affichée. Elle contient le port, par exemple `https://musique.lan:8743/`.
- La déclarer telle quelle (caractère pour caractère) dans le tableau de bord Spotify, avec *Web API* et *Web Playback SDK* cochés. Je n'ai pas vérifié que Spotify accepte une adresse HTTPS avec un port : si le tableau de bord la refuse, essaie `HTTPS_PORT=443`.
- Coller le Client ID dans l'application, puis « Se connecter à Spotify ».

## Modifier l'application

Éditer `site/index.html` : la modification se voit au rechargement (pas de cache pendant les essais).

## Pour l'essai GitHub Pages

Mettre le **contenu** de `site/` (`index.html` et `icon.png`) à la racine d'un dépôt, puis activer Pages (Settings › Pages › branche `main`, dossier `/ (root)`). L'adresse sera `https://<compte>.github.io/<dépôt>/` : c'est celle à déclarer chez Spotify, toujours en recopiant celle affichée dans Réglages.
