<!-- markdownlint-disable MD013 -->

# Imagem Docker do EaglercraftX Server

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
[Español](README.es.md) |
**Português** |
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

Este repositório publica uma imagem Docker para servidores EaglercraftX. A
imagem é criada a partir do código-fonte upstream mais recente em
[mirrorvim/eaglerX-1.8-server][upstream] e publicada no GitHub
Container Registry como
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

O projeto upstream empacota clientes de navegador, acesso WebSocket
Bungee/Waterfall e runtimes Paper em uma única imagem. Na inicialização do
contêiner, `MINECRAFT_VERSION` seleciona o runtime ativo.

## O Que Esta Imagem Fornece

| Item | Valor |
| --- | --- |
| Imagem | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Fonte de build | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Versões de servidor compatíveis | Paper `1.8.8` and Paper `1.12.2` |
| Seletor de versão | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Entrada de jogo e navegador | Port `5200` |
| Ponte admin e RCON | Port `5201`, enabled with `RCON_PASSWORD` |
| Caminho de persistência recomendado | `/eaglerX-1.8-server` |

## Implantar No Sealos

O template da Sealos App Store implanta esta imagem como um servidor com
estado, com acesso WebSocket, endpoint HTTP/admin e armazenamento persistente
para Overworld, Nether e The End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Abra o [template EaglerCraft Server][sealos-template].
2. Clique em "Deploy Now".
3. Aguarde a conclusão da implantação no Sealos.
4. Use o endpoint 5200 para jogo no navegador e entrada do servidor multiplayer.
5. Use o endpoint 5201 para a superfície HTTP/admin.

O código-fonte do template é mantido em
[`labring-actions/templates`][sealos-template-source].

## Início Rápido Com Docker

Baixe a imagem publicada:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Execute um servidor Paper 1.12.2 com dados persistentes e painel de
administração:

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

Execute um servidor Paper 1.8.8:

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

Abra o cliente no navegador em:

```text
http://localhost:5200
```

Abra o painel de administração em:

```text
http://localhost:5201/admin
```

Use o valor de `RCON_PASSWORD` quando o painel de administração solicitar a
senha.

## Comportamento Em Tempo De Execução

A imagem contém as duas árvores de versão:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

Na inicialização, `script/start_server.sh` valida `MINECRAFT_VERSION`, cria os
links simbólicos ativos, grava `server/eula.txt` e configura RCON. Em seguida,
inicia Bungee/Waterfall e Paper sob `tmux`.

Para `MINECRAFT_VERSION=1.12`, os caminhos ativos ficam:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

Para `MINECRAFT_VERSION=1.8`, os caminhos ativos ficam:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Execute contêineres separados com diretórios de host separados ao servir as
duas versões ao mesmo tempo:

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

O segundo contêiner fica disponível em `http://localhost:5300` e
`http://localhost:5301/admin`.

## Configuração

| Variável | Obrigatório | Descrição |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Sim | Seleciona `1.8` ou `1.12`. |
| `RCON_PASSWORD` | Não | Habilita RCON e a API admin na porta `5201`. |
| `SERVER_DATA_DIR` | Não | Caminho legado de persistência somente para mundos. |
| `ADMIN_AUTH_SECRET` | Não | Substitui o segredo de assinatura do token admin. |
| `ADMIN_AUTH_TOKEN_TTL` | Não | Define a duração do token admin em segundos. |
| `DYNMAP_HOST` | Não | Host opcional do proxy Dynmap. Padrão: `127.0.0.1`. |
| `DYNMAP_PORT` | Não | Porta opcional do proxy Dynmap. Padrão: `8123`. |

## Persistência

Monte `/eaglerX-1.8-server` para preservar mundos, plugins, configuração do
servidor e assets do navegador. Quando o diretório montado está vazio, o
contêiner o inicializa a partir da imagem antes de iniciar o runtime
selecionado.

Layout de host recomendado:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Mantenha cada versão em seu próprio diretório porque os caminhos ativos
`server/` e `web/` são links simbólicos específicos de versão criados na
inicialização.

## Build A Partir Do Upstream

Crie localmente o mesmo tipo de imagem a partir do repositório upstream no
Gitee:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Execute a imagem criada localmente:

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

O workflow do GitHub Actions deste repositório publica a imagem GHCR usada por
implantações downstream.

## Solução De Problemas

### O contêiner sai imediatamente

Defina `MINECRAFT_VERSION` como um dos valores compatíveis:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### O painel de administração não aceita comandos

Inicie o contêiner com `RCON_PASSWORD` e exponha a porta `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Os dados do mundo desaparecem após reiniciar

Monte o diretório runtime completo:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Use um caminho de host estável para cada versão do servidor.

### Executar as duas versões ao mesmo tempo gera conflito de portas

Mapeie o segundo contêiner para portas de host diferentes:

```bash
-p 5300:5200 -p 5301:5201
```

## Fontes E Créditos

- Projeto de servidor upstream: [mirrorvim/eaglerX-1.8-server][upstream]
- Imagem publicada: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Trabalho original de Eaglercraft e EaglercraftX: lax1dude (Calder Young)
- Trabalho de Eaglercraft server: ayunami2000

## Licença

Este repositório é licenciado sob a [MIT License][license]. Revise o projeto
upstream e os componentes incluídos para suas próprias licenças antes de
redistribuí-los.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
