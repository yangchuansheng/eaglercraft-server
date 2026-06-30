<!-- markdownlint-disable MD013 -->

# EaglercraftX Server Docker 镜像

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
[Español](README.es.md) |
[Português](README.pt.md) |
[Deutsch](README.de.md) |
[Français](README.fr.md) |
**简体中文** |
[繁體中文](README.zh-Hant.md) |
[한국어](README.ko.md) |
[日本語](README.ja.md) |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

此仓库发布用于 EaglercraftX 服务器的 Docker 镜像。 镜像基于
[mirrorvim/eaglerX-1.8-server][upstream] 的最新上游源码构建， 并以
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package] 发布到 GitHub
Container Registry。

上游项目把浏览器客户端、Bungee/Waterfall WebSocket 访问和 Paper 服务端运行时
打包到同一个镜像中。容器启动时，`MINECRAFT_VERSION` 会选择当前运行时。

## 此镜像提供什么

| 项目 | 值 |
| --- | --- |
| 镜像 | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| 构建来源 | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| 支持的服务器版本 | Paper `1.8.8` and Paper `1.12.2` |
| 版本选择器 | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| 游戏和浏览器入口 | Port `5200` |
| Admin 和 RCON 桥接 | Port `5201`, enabled with `RCON_PASSWORD` |
| 推荐持久化路径 | `/eaglerX-1.8-server` |

## 部署到 Sealos

Sealos App Store 模板会把此镜像部署为有状态服务器，提供 WebSocket 访问、 HTTP/admin 端点，并为
Overworld、Nether 和 The End 提供持久化存储。

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. 打开 [EaglerCraft Server 模板][sealos-template]。
2. 点击 "Deploy Now"。
3. 等待 Sealos 部署完成。
4. 使用 5200 端点进行浏览器游戏和多人服务器入口访问。
5. 使用 5201 端点访问 HTTP/admin 界面。

模板源码维护在
[`labring-actions/templates`][sealos-template-source]。

## Docker 快速开始

拉取已发布镜像：

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

运行带持久化数据和管理面板的 Paper 1.12.2 服务器：

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

运行 Paper 1.8.8 服务器：

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

在此打开浏览器客户端：

```text
http://localhost:5200
```

在此打开管理面板：

```text
http://localhost:5201/admin
```

当管理面板提示输入密码时，使用 `RCON_PASSWORD` 的值。

## 运行时行为

镜像包含两个版本目录树：

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

启动时，`script/start_server.sh` 会校验 `MINECRAFT_VERSION`，创建活动 软链接，写入
`server/eula.txt` 并配置 RCON。随后它会在 `tmux` 下启动 Bungee/Waterfall 和 Paper。

当 `MINECRAFT_VERSION=1.12` 时，活动路径为：

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

当 `MINECRAFT_VERSION=1.8` 时，活动路径为：

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

同时提供两个版本时，请使用不同宿主机目录运行独立容器：

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

第二个容器可通过 `http://localhost:5300` 和 `http://localhost:5301/admin` 访问。

## 配置

| 变量 | 必填 | 说明 |
| --- | --- | --- |
| `MINECRAFT_VERSION` | 是 | 选择 `1.8` 或 `1.12`。 |
| `RCON_PASSWORD` | 否 | 在 `5201` 端口启用 RCON 和 admin API。 |
| `SERVER_DATA_DIR` | 否 | 旧版仅世界数据持久化路径。 |
| `ADMIN_AUTH_SECRET` | 否 | 覆盖 admin token 签名密钥。 |
| `ADMIN_AUTH_TOKEN_TTL` | 否 | 设置 admin token 生命周期，单位为秒。 |
| `DYNMAP_HOST` | 否 | 可选 Dynmap 代理主机。默认：`127.0.0.1`。 |
| `DYNMAP_PORT` | 否 | 可选 Dynmap 代理端口。默认：`8123`。 |

## 持久化

挂载 `/eaglerX-1.8-server` 可持久保存世界、插件、服务器配置和浏览器
资源。当挂载目录为空时，容器会先从镜像初始化该目录，再启动选定运行时。

推荐宿主机布局：

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

每个版本使用独立目录，因为活动 `server/` 和 `web/` 路径是启动时创建的 版本专属软链接。

## 从上游构建

从 Gitee 上游仓库本地构建同类型镜像：

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

运行本地构建的镜像：

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

此仓库的 GitHub Actions workflow 会发布下游部署使用的 GHCR 镜像。

## 故障排查

### 容器立即退出

将 `MINECRAFT_VERSION` 设置为支持的值之一：

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### 管理面板不接受命令

使用 `RCON_PASSWORD` 启动容器并暴露 `5201` 端口：

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### 重启后世界数据消失

挂载完整运行目录：

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

为每个服务器版本使用稳定的宿主机路径。

### 同时运行两个版本会产生端口冲突

将第二个容器映射到不同宿主机端口：

```bash
-p 5300:5200 -p 5301:5201
```

## 来源和致谢

- 上游服务器项目: [mirrorvim/eaglerX-1.8-server][upstream]
- 已发布镜像: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Eaglercraft 和 EaglercraftX 原始工作：lax1dude (Calder Young)
- Eaglercraft server 工作：ayunami2000

## 许可证

此仓库使用 [MIT License][license] 授权。重新分发前，请查看上游项目和打包组件 各自的许可证。

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
