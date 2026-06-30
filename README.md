# EaglercraftX Server Docker Image

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

**English** | [Español](readmes/README.es.md) |
[Português](readmes/README.pt.md) | [Deutsch](readmes/README.de.md) |
[Français](readmes/README.fr.md) | [简体中文](readmes/README.zh.md) |
[繁體中文](readmes/README.zh-Hant.md) | [한국어](readmes/README.ko.md) |
[日本語](readmes/README.ja.md) | [العربية](readmes/README.ar.md) |
[Русский](readmes/README.ru.md) | [Українська](readmes/README.uk.md) |
[Türkçe](readmes/README.tr.md)

<!-- README-I18N:END -->

This repository publishes a Docker image for EaglercraftX servers.
The image is built from the latest upstream source at
[mirrorvim/eaglerX-1.8-server][upstream] and is published to GitHub
Container Registry as [`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

The upstream project packages browser clients, Bungee/Waterfall WebSocket
access, and Paper server runtimes into one image. At container startup,
`MINECRAFT_VERSION` selects the active runtime.

## What This Image Provides

| Item | Value |
| --- | --- |
| Image | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Build source | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Supported server versions | Paper `1.8.8` and Paper `1.12.2` |
| Version selector | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Game and browser entry | Port `5200` |
| Admin and RCON bridge | Port `5201`, enabled with `RCON_PASSWORD` |
| Recommended persistence path | `/eaglerX-1.8-server` |

## Deploy On Sealos

The Sealos App Store template deploys this image as a stateful server with
WebSocket access, an HTTP/admin endpoint, and persistent storage for the
Overworld, Nether, and The End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Open the [EaglerCraft Server template][sealos-template].
2. Click "Deploy Now".
3. Wait for the Sealos deployment to finish.
4. Use the 5200 endpoint for browser gameplay and multiplayer server entry.
5. Use the 5201 endpoint for the HTTP/admin surface.

The template source is maintained in
[`labring-actions/templates`][sealos-template-source].

## Docker Quick Start

Pull the published image:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Run a Paper 1.12.2 server with persistent data and the admin panel:

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

Run a Paper 1.8.8 server:

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

Open the browser client at:

```text
http://localhost:5200
```

Open the admin panel at:

```text
http://localhost:5201/admin
```

Use the value of `RCON_PASSWORD` when the admin panel prompts for a password.

## Runtime Behavior

The image contains both version trees:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

At startup, `script/start_server.sh` validates `MINECRAFT_VERSION`, creates the
active symlinks, writes `server/eula.txt`, and configures RCON. It then starts
Bungee/Waterfall and Paper under `tmux`.

For `MINECRAFT_VERSION=1.12`, the active paths become:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

For `MINECRAFT_VERSION=1.8`, the active paths become:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Run separate containers with separate host directories when serving both
versions at the same time:

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

The second container is available at `http://localhost:5300` and `http://localhost:5301/admin`.

## Configuration

| Variable | Required | Description |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Yes | Selects `1.8` or `1.12`. |
| `RCON_PASSWORD` | No | Enables RCON and the admin API on port `5201`. |
| `SERVER_DATA_DIR` | No | Legacy world-only persistence path. |
| `ADMIN_AUTH_SECRET` | No | Overrides the admin token signing secret. |
| `ADMIN_AUTH_TOKEN_TTL` | No | Sets the admin token lifetime in seconds. |
| `DYNMAP_HOST` | No | Optional Dynmap proxy host. Default: `127.0.0.1`. |
| `DYNMAP_PORT` | No | Optional Dynmap proxy port. Default: `8123`. |

## Persistence

Mount `/eaglerX-1.8-server` for durable worlds, plugins, server configuration,
and browser assets. When the mounted directory is empty, the container
initializes it from the image before starting the selected runtime.

Recommended host layout:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Keep each version in its own directory because the active `server/` and `web/`
paths are version-specific symlinks created at startup.

## Build From Upstream

Build the same style of image locally from the upstream Gitee repository:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Run the locally built image:

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

This repository's GitHub Actions workflow publishes the GHCR image used by
downstream deployments.

## Troubleshooting

### Container exits immediately

Set `MINECRAFT_VERSION` to one of the supported values:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### Admin panel does not accept commands

Start the container with `RCON_PASSWORD` and expose port `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### World data disappears after restart

Mount the full runtime directory:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Use a stable host path for each server version.

### Running both versions at the same time conflicts on ports

Map the second container to different host ports:

```bash
-p 5300:5200 -p 5301:5201
```

## Sources And Credits

- Upstream server project: [mirrorvim/eaglerX-1.8-server][upstream]
- Published image: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Original Eaglercraft and EaglercraftX work: lax1dude (Calder Young)
- Eaglercraft server work: ayunami2000

## License

This repository is licensed under the [MIT License][license]. Review the
upstream project and bundled components for their own licenses before
redistribution.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
