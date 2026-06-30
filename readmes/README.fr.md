<!-- markdownlint-disable MD013 -->

# Image Docker EaglercraftX Server

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
[Español](README.es.md) |
[Português](README.pt.md) |
[Deutsch](README.de.md) |
**Français** |
[简体中文](README.zh.md) |
[繁體中文](README.zh-Hant.md) |
[한국어](README.ko.md) |
[日本語](README.ja.md) |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

Ce dépôt publie une image Docker pour les serveurs EaglercraftX. L’image est
construite depuis la source upstream la plus récente de
[mirrorvim/eaglerX-1.8-server][upstream] et publiée dans GitHub
Container Registry sous le nom
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

Le projet upstream regroupe les clients navigateur, l’accès WebSocket
Bungee/Waterfall et les runtimes Paper dans une seule image. Au démarrage du
conteneur, `MINECRAFT_VERSION` sélectionne le runtime actif.

## Ce Que Fournit Cette Image

| Élément | Valeur |
| --- | --- |
| Image | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Source de build | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Versions serveur prises en charge | Paper `1.8.8` and Paper `1.12.2` |
| Sélecteur de version | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Entrée jeu et navigateur | Port `5200` |
| Pont admin et RCON | Port `5201`, enabled with `RCON_PASSWORD` |
| Chemin de persistance recommandé | `/eaglerX-1.8-server` |

## Déployer Sur Sealos

Le modèle Sealos App Store déploie cette image comme un serveur avec état,
avec accès WebSocket, endpoint HTTP/admin et stockage persistant pour
l’Overworld, le Nether et The End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Ouvrez le [modèle EaglerCraft Server][sealos-template].
2. Cliquez sur "Deploy Now".
3. Attendez la fin du déploiement Sealos.
4. Utilisez l’endpoint 5200 pour le jeu dans le navigateur et l’entrée serveur multijoueur.
5. Utilisez l’endpoint 5201 pour la surface HTTP/admin.

La source du modèle est maintenue dans
[`labring-actions/templates`][sealos-template-source].

## Démarrage Rapide Docker

Téléchargez l’image publiée:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Lancez un serveur Paper 1.12.2 avec données persistantes et panneau admin:

```bash
docker run -d \
  --name eaglerx-1-12 \
  -p 5200:5200 \
  -p 5201:5201 \
  -v "$PWD/eagler-data-1.12:/eaglerX-1.8-server" \
  -e MINECRAFT_VERSION=1.12 \
  -e RCON_PASSWORD=change-this-password \
  ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Lancez un serveur Paper 1.8.8:

```bash
docker run -d \
  --name eaglerx-1-8 \
  -p 5200:5200 \
  -p 5201:5201 \
  -v "$PWD/eagler-data-1.8:/eaglerX-1.8-server" \
  -e MINECRAFT_VERSION=1.8 \
  -e RCON_PASSWORD=change-this-password \
  ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Ouvrez le client navigateur à:

```text
http://localhost:5200
```

Ouvrez le panneau admin à:

```text
http://localhost:5201/admin
```

Utilisez la valeur de `RCON_PASSWORD` quand le panneau admin demande un mot de
passe.

## Comportement À L’exécution

L’image contient les deux arbres de version:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

Au démarrage, `script/start_server.sh` valide `MINECRAFT_VERSION`, crée les
liens symboliques actifs, écrit `server/eula.txt` et configure RCON. Il lance
ensuite Bungee/Waterfall et Paper sous `tmux`.

Pour `MINECRAFT_VERSION=1.12`, les chemins actifs deviennent:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

Pour `MINECRAFT_VERSION=1.8`, les chemins actifs deviennent:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Lancez des conteneurs séparés avec des répertoires hôte séparés lorsque vous
servez les deux versions en même temps:

```bash
docker run -d \
  --name eaglerx-1-12 \
  -p 5200:5200 \
  -p 5201:5201 \
  -v /data/eagler-1.12:/eaglerX-1.8-server \
  -e MINECRAFT_VERSION=1.12 \
  -e RCON_PASSWORD=change-this-password \
  ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1

docker run -d \
  --name eaglerx-1-8 \
  -p 5300:5200 \
  -p 5301:5201 \
  -v /data/eagler-1.8:/eaglerX-1.8-server \
  -e MINECRAFT_VERSION=1.8 \
  -e RCON_PASSWORD=change-this-password \
  ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Le second conteneur est disponible à `http://localhost:5300` et
`http://localhost:5301/admin`.

## Configuration

| Variable | Requis | Description |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Oui | Sélectionne `1.8` ou `1.12`. |
| `RCON_PASSWORD` | Non | Active RCON et l’API admin sur le port `5201`. |
| `SERVER_DATA_DIR` | Non | Ancien chemin de persistance réservé aux mondes. |
| `ADMIN_AUTH_SECRET` | Non | Remplace le secret de signature du jeton admin. |
| `ADMIN_AUTH_TOKEN_TTL` | Non | Définit la durée de vie du jeton admin en secondes. |
| `DYNMAP_HOST` | Non | Hôte proxy Dynmap facultatif. Par défaut: `127.0.0.1`. |
| `DYNMAP_PORT` | Non | Port proxy Dynmap facultatif. Par défaut: `8123`. |

## Persistance

Montez `/eaglerX-1.8-server` pour conserver les mondes, plugins, la
configuration serveur et les assets navigateur. Quand le répertoire monté est
vide, le conteneur l’initialise depuis l’image avant de démarrer le runtime
choisi.

Disposition hôte recommandée:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Gardez chaque version dans son propre répertoire, car les chemins actifs
`server/` et `web/` sont des liens symboliques propres à la version créés au
démarrage.

## Construire Depuis Upstream

Construisez localement le même type d’image depuis le dépôt Gitee upstream:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Lancez l’image construite localement:

```bash
docker run -d \
  --name eaglerx-local \
  -p 5200:5200 \
  -p 5201:5201 \
  -v "$PWD/eagler-data:/eaglerX-1.8-server" \
  -e MINECRAFT_VERSION=1.12 \
  -e RCON_PASSWORD=change-this-password \
  ghcr.io/yangchuansheng/eaglerx1.8server:local
```

Le workflow GitHub Actions de ce dépôt publie l’image GHCR utilisée par les
déploiements downstream.

## Dépannage

### Le conteneur s’arrête immédiatement

Définissez `MINECRAFT_VERSION` sur l’une des valeurs prises en charge:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### Le panneau admin n’accepte pas les commandes

Lancez le conteneur avec `RCON_PASSWORD` et exposez le port `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Les données du monde disparaissent après redémarrage

Montez le répertoire runtime complet:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Utilisez un chemin hôte stable pour chaque version serveur.

### Exécuter les deux versions en même temps crée un conflit de ports

Mappez le second conteneur vers des ports hôte différents:

```bash
-p 5300:5200 -p 5301:5201
```

## Sources Et Crédits

- Projet serveur upstream: [mirrorvim/eaglerX-1.8-server][upstream]
- Image publiée: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Travail original Eaglercraft et EaglercraftX: lax1dude (Calder Young)
- Travail Eaglercraft server: ayunami2000

## Licence

Ce dépôt est sous [MIT License][license]. Consultez le projet upstream et les
composants inclus pour leurs propres licences avant toute redistribution.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
