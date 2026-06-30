<!-- markdownlint-disable MD013 -->

# Imagen Docker de EaglercraftX Server

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
**Español** |
[Português](README.pt.md) |
[Deutsch](README.de.md) |
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

Este repositorio publica una imagen Docker para servidores EaglercraftX. La
imagen se construye desde el código fuente upstream más reciente de
[mirrorvim/eaglerX-1.8-server][upstream] y se publica en GitHub
Container Registry como
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

El proyecto upstream empaqueta clientes de navegador, acceso WebSocket
Bungee/Waterfall y runtimes Paper en una sola imagen. Al iniciar el
contenedor, `MINECRAFT_VERSION` selecciona el runtime activo.

## Qué Proporciona Esta Imagen

| Elemento | Valor |
| --- | --- |
| Imagen | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Fuente de compilación | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Versiones de servidor compatibles | Paper `1.8.8` and Paper `1.12.2` |
| Selector de versión | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Entrada de juego y navegador | Port `5200` |
| Puente admin y RCON | Port `5201`, enabled with `RCON_PASSWORD` |
| Ruta de persistencia recomendada | `/eaglerX-1.8-server` |

## Desplegar En Sealos

La plantilla de Sealos App Store despliega esta imagen como un servidor con
estado, con acceso WebSocket, un endpoint HTTP/admin y almacenamiento
persistente para el Overworld, Nether y The End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Abre la [plantilla de EaglerCraft Server][sealos-template].
2. Haz clic en "Deploy Now".
3. Espera a que termine el despliegue en Sealos.
4. Usa el endpoint 5200 para el juego en navegador y la entrada del servidor multijugador.
5. Usa el endpoint 5201 para la superficie HTTP/admin.

El código fuente de la plantilla se mantiene en
[`labring-actions/templates`][sealos-template-source].

## Inicio Rápido Con Docker

Descarga la imagen publicada:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Ejecuta un servidor Paper 1.12.2 con datos persistentes y panel de
administración:

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

Ejecuta un servidor Paper 1.8.8:

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

Abre el cliente de navegador en:

```text
http://localhost:5200
```

Abre el panel de administración en:

```text
http://localhost:5201/admin
```

Usa el valor de `RCON_PASSWORD` cuando el panel de administración pida la
contraseña.

## Comportamiento En Tiempo De Ejecución

La imagen contiene ambos árboles de versión:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

Al iniciar, `script/start_server.sh` valida `MINECRAFT_VERSION`, crea los
enlaces simbólicos activos, escribe `server/eula.txt` y configura RCON. Luego
inicia Bungee/Waterfall y Paper bajo `tmux`.

Para `MINECRAFT_VERSION=1.12`, las rutas activas son:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

Para `MINECRAFT_VERSION=1.8`, las rutas activas son:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Ejecuta contenedores separados con directorios de host separados cuando sirvas
ambas versiones al mismo tiempo:

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

El segundo contenedor está disponible en `http://localhost:5300` y
`http://localhost:5301/admin`.

## Configuración

| Variable | Requerida | Descripción |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Sí | Selecciona `1.8` o `1.12`. |
| `RCON_PASSWORD` | No | Habilita RCON y la API admin en el puerto `5201`. |
| `SERVER_DATA_DIR` | No | Ruta heredada de persistencia solo para mundos. |
| `ADMIN_AUTH_SECRET` | No | Sobrescribe el secreto de firma del token admin. |
| `ADMIN_AUTH_TOKEN_TTL` | No | Define la duración del token admin en segundos. |
| `DYNMAP_HOST` | No | Host opcional del proxy Dynmap. Valor predeterminado: `127.0.0.1`. |
| `DYNMAP_PORT` | No | Puerto opcional del proxy Dynmap. Valor predeterminado: `8123`. |

## Persistencia

Monta `/eaglerX-1.8-server` para conservar mundos, plugins, configuración del
servidor y assets del navegador. Cuando el directorio montado está vacío, el
contenedor lo inicializa desde la imagen antes de iniciar el runtime
seleccionado.

Distribución recomendada del host:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Mantén cada versión en su propio directorio porque las rutas activas `server/`
y `web/` son enlaces simbólicos específicos de versión creados al inicio.

## Compilar Desde Upstream

Compila localmente el mismo tipo de imagen desde el repositorio upstream de
Gitee:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Ejecuta la imagen compilada localmente:

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

El workflow de GitHub Actions de este repositorio publica la imagen GHCR usada
por los despliegues downstream.

## Solución De Problemas

### El contenedor sale inmediatamente

Configura `MINECRAFT_VERSION` con uno de los valores admitidos:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### El panel de administración no acepta comandos

Inicia el contenedor con `RCON_PASSWORD` y expón el puerto `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Los datos del mundo desaparecen tras reiniciar

Monta el directorio runtime completo:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Usa una ruta de host estable para cada versión del servidor.

### Ejecutar ambas versiones al mismo tiempo crea conflictos de puertos

Mapea el segundo contenedor a puertos de host diferentes:

```bash
-p 5300:5200 -p 5301:5201
```

## Fuentes Y Créditos

- Proyecto de servidor upstream: [mirrorvim/eaglerX-1.8-server][upstream]
- Imagen publicada: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Trabajo original de Eaglercraft y EaglercraftX: lax1dude (Calder Young)
- Trabajo de Eaglercraft server: ayunami2000

## Licencia

Este repositorio tiene licencia bajo la [MIT License][license]. Revisa el
proyecto upstream y los componentes incluidos para conocer sus propias
licencias antes de redistribuirlos.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
