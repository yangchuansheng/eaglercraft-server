<!-- markdownlint-disable MD013 -->

# EaglercraftX Server Docker-Image

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
[Español](README.es.md) |
[Português](README.pt.md) |
**Deutsch** |
[Français](README.fr.md) |
[简体中文](README.zh.md) |
[繁體中文](README.zh-Hant.md) |
[한국어](README.ko.md) |
[日本語](README.ja.md) |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

Dieses Repository veröffentlicht ein Docker-Image für EaglercraftX-Server. Das
Image wird aus der neuesten Upstream-Quelle unter
[mirrorvim/eaglerX-1.8-server][upstream] gebaut und in GitHub
Container Registry als
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package] veröffentlicht.

Das Upstream-Projekt bündelt Browser-Clients,
Bungee/Waterfall-WebSocket-Zugriff und Paper-Laufzeiten in einem Image. Beim
Containerstart wählt `MINECRAFT_VERSION` die aktive Laufzeit aus.

## Was Dieses Image Bereitstellt

| Element | Wert |
| --- | --- |
| Image | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Build-Quelle | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Unterstützte Serverversionen | Paper `1.8.8` and Paper `1.12.2` |
| Versionsauswahl | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Spiel- und Browsereinstieg | Port `5200` |
| Admin- und RCON-Brücke | Port `5201`, enabled with `RCON_PASSWORD` |
| Empfohlener Persistenzpfad | `/eaglerX-1.8-server` |

## Auf Sealos Bereitstellen

Das Sealos App Store Template stellt dieses Image als zustandsbehafteten
Server mit WebSocket-Zugriff, HTTP/Admin-Endpunkt und persistenter Speicherung
für Overworld, Nether und The End bereit.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Öffne das [EaglerCraft Server Template][sealos-template].
2. Klicke auf "Deploy Now".
3. Warte, bis die Sealos-Bereitstellung abgeschlossen ist.
4. Nutze den 5200-Endpunkt für Browser-Gameplay und den Multiplayer-Servereintrag.
5. Nutze den 5201-Endpunkt für die HTTP/Admin-Oberfläche.

Die Template-Quelle wird in
[`labring-actions/templates`][sealos-template-source] gepflegt.

## Docker-Schnellstart

Ziehe das veröffentlichte Image:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Starte einen Paper 1.12.2 Server mit persistenten Daten und Admin-Panel:

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

Starte einen Paper 1.8.8 Server:

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

Öffne den Browser-Client unter:

```text
http://localhost:5200
```

Öffne das Admin-Panel unter:

```text
http://localhost:5201/admin
```

Verwende den Wert von `RCON_PASSWORD`, wenn das Admin-Panel nach einem
Passwort fragt.

## Laufzeitverhalten

Das Image enthält beide Versionsbäume:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

Beim Start validiert `script/start_server.sh` `MINECRAFT_VERSION`, erstellt
die aktiven Symlinks, schreibt `server/eula.txt` und konfiguriert RCON. Danach
startet es Bungee/Waterfall und Paper unter `tmux`.

Für `MINECRAFT_VERSION=1.12` werden die aktiven Pfade:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

Für `MINECRAFT_VERSION=1.8` werden die aktiven Pfade:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Starte getrennte Container mit getrennten Host-Verzeichnissen, wenn beide
Versionen gleichzeitig bereitgestellt werden:

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

Der zweite Container ist unter `http://localhost:5300` und
`http://localhost:5301/admin` erreichbar.

## Konfiguration

| Variable | Erforderlich | Beschreibung |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Ja | Wählt `1.8` oder `1.12` aus. |
| `RCON_PASSWORD` | Nein | Aktiviert RCON und die Admin-API auf Port `5201`. |
| `SERVER_DATA_DIR` | Nein | Legacy-Persistenzpfad nur für Welten. |
| `ADMIN_AUTH_SECRET` | Nein | Überschreibt das Signatur-Secret für Admin-Tokens. |
| `ADMIN_AUTH_TOKEN_TTL` | Nein | Setzt die Lebensdauer des Admin-Tokens in Sekunden. |
| `DYNMAP_HOST` | Nein | Optionaler Dynmap-Proxy-Host. Standard: `127.0.0.1`. |
| `DYNMAP_PORT` | Nein | Optionaler Dynmap-Proxy-Port. Standard: `8123`. |

## Persistenz

Binde `/eaglerX-1.8-server` ein, um Welten, Plugins, Serverkonfiguration und
Browser-Assets dauerhaft zu speichern. Wenn das eingehängte Verzeichnis leer
ist, initialisiert der Container es aus dem Image vor dem Start der gewählten
Laufzeit.

Empfohlene Host-Struktur:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Halte jede Version in einem eigenen Verzeichnis, da die aktiven Pfade
`server/` und `web/` versionsspezifische Symlinks sind, die beim Start
erstellt werden.

## Aus Upstream Bauen

Baue denselben Image-Typ lokal aus dem Upstream-Gitee-Repository:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Starte das lokal gebaute Image:

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

Der GitHub-Actions-Workflow dieses Repositorys veröffentlicht das GHCR-Image,
das von Downstream-Bereitstellungen genutzt wird.

## Fehlerbehebung

### Der Container beendet sich sofort

Setze `MINECRAFT_VERSION` auf einen der unterstützten Werte:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### Das Admin-Panel nimmt keine Befehle an

Starte den Container mit `RCON_PASSWORD` und veröffentliche Port `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Weltdaten verschwinden nach einem Neustart

Binde das vollständige Laufzeitverzeichnis ein:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Verwende einen stabilen Host-Pfad für jede Serverversion.

### Beide Versionen gleichzeitig verursachen Portkonflikte

Mappe den zweiten Container auf andere Host-Ports:

```bash
-p 5300:5200 -p 5301:5201
```

## Quellen Und Credits

- Upstream-Serverprojekt: [mirrorvim/eaglerX-1.8-server][upstream]
- Veröffentlichtes Image: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Originalarbeit von Eaglercraft und EaglercraftX: lax1dude (Calder Young)
- Eaglercraft server Arbeit: ayunami2000

## Lizenz

Dieses Repository ist unter der [MIT License][license] lizenziert. Prüfe das
Upstream-Projekt und die gebündelten Komponenten auf ihre eigenen Lizenzen,
bevor du sie weiterverteilst.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
