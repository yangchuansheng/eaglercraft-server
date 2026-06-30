<!-- markdownlint-disable MD013 -->

# EaglercraftX Server Docker 이미지

[![Container package][container-badge]][container-package]
[![License][license-badge]][license]

[![Deploy on Sealos][sealos-badge]][sealos-template]

<!-- README-I18N:START -->

[English](../README.md) |
[Español](README.es.md) |
[Português](README.pt.md) |
[Deutsch](README.de.md) |
[Français](README.fr.md) |
[简体中文](README.zh.md) |
[繁體中文](README.zh-Hant.md) |
**한국어** |
[日本語](README.ja.md) |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

이 저장소는 EaglercraftX 서버용 Docker 이미지를 게시합니다. 이 이미지는
[mirrorvim/eaglerX-1.8-server][upstream]의 최신 upstream 소스에서 빌드되며
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package] 이름으로 GitHub
Container Registry에 게시됩니다.

upstream 프로젝트는 브라우저 클라이언트, Bungee/Waterfall WebSocket 접근, Paper 런타임을 하나의 이미지로
패키징합니다. 컨테이너가 시작될 때 `MINECRAFT_VERSION`이 활성 런타임을 선택합니다.

## 이 이미지가 제공하는 것

| 항목 | 값 |
| --- | --- |
| 이미지 | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| 빌드 소스 | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| 지원 서버 버전 | Paper `1.8.8` and Paper `1.12.2` |
| 버전 선택기 | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| 게임 및 브라우저 진입점 | Port `5200` |
| Admin 및 RCON 브리지 | Port `5201`, enabled with `RCON_PASSWORD` |
| 권장 영구 저장 경로 | `/eaglerX-1.8-server` |

## Sealos에 배포

Sealos App Store 템플릿은 이 이미지를 WebSocket 접근, HTTP/admin 엔드포인트, Overworld,
Nether, The End용 영구 저장소를 갖춘 상태 저장 서버로 배포합니다.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. [EaglerCraft Server 템플릿][sealos-template]을 엽니다.
2. "Deploy Now"를 클릭합니다.
3. Sealos 배포가 완료될 때까지 기다립니다.
4. 브라우저 플레이와 멀티플레이어 서버 진입에는 5200 엔드포인트를 사용합니다.
5. HTTP/admin 화면에는 5201 엔드포인트를 사용합니다.

템플릿 소스는
[`labring-actions/templates`][sealos-template-source]에서 관리됩니다.

## Docker 빠른 시작

게시된 이미지를 가져옵니다:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

영구 데이터와 admin 패널이 있는 Paper 1.12.2 서버를 실행합니다:

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

Paper 1.8.8 서버를 실행합니다:

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

브라우저 클라이언트를 엽니다:

```text
http://localhost:5200
```

admin 패널을 엽니다:

```text
http://localhost:5201/admin
```

admin 패널에서 비밀번호를 요청하면 `RCON_PASSWORD` 값을 사용합니다.

## 런타임 동작

이미지에는 두 버전 트리가 포함되어 있습니다:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

시작 시 `script/start_server.sh`는 `MINECRAFT_VERSION`을 검증하고 활성 심볼릭 링크를 만들며
`server/eula.txt`를 쓰고 RCON을 구성합니다. 그런 다음 `tmux` 아래에서 Bungee/Waterfall과 Paper를
시작합니다.

`MINECRAFT_VERSION=1.12`일 때 활성 경로는 다음과 같습니다:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

`MINECRAFT_VERSION=1.8`일 때 활성 경로는 다음과 같습니다:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

두 버전을 동시에 제공하려면 별도 호스트 디렉터리를 사용하는 별도 컨테이너를 실행합니다:

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

두 번째 컨테이너는 `http://localhost:5300` 및 `http://localhost:5301/admin`에서 사용할 수
있습니다.

## 구성

| 변수 | 필수 | 설명 |
| --- | --- | --- |
| `MINECRAFT_VERSION` | 예 | `1.8` 또는 `1.12`를 선택합니다. |
| `RCON_PASSWORD` | 아니요 | `5201` 포트에서 RCON 및 admin API를 활성화합니다. |
| `SERVER_DATA_DIR` | 아니요 | 기존 world 전용 영구 저장 경로입니다. |
| `ADMIN_AUTH_SECRET` | 아니요 | admin 토큰 서명 비밀값을 재정의합니다. |
| `ADMIN_AUTH_TOKEN_TTL` | 아니요 | admin 토큰 수명을 초 단위로 설정합니다. |
| `DYNMAP_HOST` | 아니요 | 선택적 Dynmap 프록시 호스트. 기본값: `127.0.0.1`. |
| `DYNMAP_PORT` | 아니요 | 선택적 Dynmap 프록시 포트. 기본값: `8123`. |

## 영구 저장

world, plugin, 서버 구성, 브라우저 asset을 보존하려면 `/eaglerX-1.8-server`를 마운트합니다. 마운트된
디렉터리가 비어 있으면 컨테이너가 선택된 런타임을 시작하기 전에 이미지에서 초기화합니다.

권장 호스트 레이아웃:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

활성 `server/` 및 `web/` 경로는 시작 시 생성되는 버전별 심볼릭 링크이므로 각 버전을 자체 디렉터리에 유지합니다.

## Upstream에서 빌드

upstream Gitee 저장소에서 같은 유형의 이미지를 로컬로 빌드합니다:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

로컬로 빌드한 이미지를 실행합니다:

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

이 저장소의 GitHub Actions workflow는 downstream 배포에서 사용하는 GHCR 이미지를 게시합니다.

## 문제 해결

### 컨테이너가 즉시 종료됨

`MINECRAFT_VERSION`을 지원되는 값 중 하나로 설정합니다:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### admin 패널이 명령을 받지 않음

`RCON_PASSWORD`로 컨테이너를 시작하고 `5201` 포트를 노출합니다:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### 재시작 후 world 데이터가 사라짐

전체 런타임 디렉터리를 마운트합니다:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

각 서버 버전에 안정적인 호스트 경로를 사용합니다.

### 두 버전을 동시에 실행하면 포트 충돌이 발생함

두 번째 컨테이너를 다른 호스트 포트에 매핑합니다:

```bash
-p 5300:5200 -p 5301:5201
```

## 출처 및 크레딧

- upstream 서버 프로젝트: [mirrorvim/eaglerX-1.8-server][upstream]
- 게시된 이미지: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Eaglercraft 및 EaglercraftX 원작: lax1dude (Calder Young)
- Eaglercraft server 작업: ayunami2000

## 라이선스

이 저장소는 [MIT License][license]로 라이선스가 부여됩니다. 재배포하기 전에 upstream 프로젝트와 번들 구성 요소의
개별 라이선스를 확인하세요.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
