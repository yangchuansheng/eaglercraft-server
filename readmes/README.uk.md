<!-- markdownlint-disable MD013 -->

# Docker-образ EaglercraftX Server

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
[한국어](README.ko.md) |
[日本語](README.ja.md) |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
**Українська** |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

Цей репозиторій публікує Docker-образ для серверів EaglercraftX. Образ
собирается из последнего upstream-исходного кода
[mirrorvim/eaglerX-1.8-server][upstream] и публикуется в GitHub
Container Registry как
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

upstream-проект объединяет браузерные клиенты, WebSocket-доступ
Bungee/Waterfall и среды Paper в одном образе. При запуске контейнера
`MINECRAFT_VERSION` выбирает активную среду.

## Що Надає Цей Образ

| Пункт | Значення |
| --- | --- |
| Образ | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Источник сборки | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Поддерживаемые версии сервера | Paper `1.8.8` and Paper `1.12.2` |
| Выбор версии | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Вход для игры и браузера | Port `5200` |
| Мост admin и RCON | Port `5201`, enabled with `RCON_PASSWORD` |
| Рекомендуемый путь хранения | `/eaglerX-1.8-server` |

## Розгортання В Sealos

Шаблон Sealos App Store разворачивает этот образ как сервер с состоянием,
WebSocket-доступом, HTTP/admin endpoint и постоянным хранилищем для Overworld,
Nether и The End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. Откройте [шаблон EaglerCraft Server][sealos-template].
2. Нажмите "Deploy Now".
3. Дождитесь завершения развертывания Sealos.
4. Используйте endpoint 5200 для браузерной игры и входа multiplayer-сервера.
5. Используйте endpoint 5201 для HTTP/admin поверхности.

Источник шаблона поддерживается в
[`labring-actions/templates`][sealos-template-source].

## Швидкий Старт Docker

Загрузите опубликованный образ:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Запустите сервер Paper 1.12.2 с постоянными данными и admin-панелью:

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

Запустите сервер Paper 1.8.8:

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

Откройте браузерный клиент по адресу:

```text
http://localhost:5200
```

Откройте admin-панель по адресу:

```text
http://localhost:5201/admin
```

Используйте значение `RCON_PASSWORD`, когда admin-панель запросит пароль.

## Поведение Во Время Работы

Образ содержит оба дерева версий:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

При запуске `script/start_server.sh` проверяет `MINECRAFT_VERSION`, создает
активные символические ссылки, записывает `server/eula.txt` и настраивает
RCON. Затем он запускает Bungee/Waterfall и Paper под `tmux`.

Для `MINECRAFT_VERSION=1.12` активные пути становятся:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

Для `MINECRAFT_VERSION=1.8` активные пути становятся:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

Запускайте отдельные контейнеры с отдельными директориями хоста, когда
обслуживаете обе версии одновременно:

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

Второй контейнер доступен по `http://localhost:5300` и
`http://localhost:5301/admin`.

## Конфігурація

| Переменная | Обов’язково | Опис |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Так | Выбирает `1.8` или `1.12`. |
| `RCON_PASSWORD` | Ні | Включает RCON и admin API на порту `5201`. |
| `SERVER_DATA_DIR` | Ні | Устаревший путь постоянного хранения только для миров. |
| `ADMIN_AUTH_SECRET` | Ні | Переопределяет секрет подписи admin-токена. |
| `ADMIN_AUTH_TOKEN_TTL` | Ні | Задает срок жизни admin-токена в секундах. |
| `DYNMAP_HOST` | Ні | Необязательный хост прокси Dynmap. По умолчанию: `127.0.0.1`. |
| `DYNMAP_PORT` | Ні | Необязательный порт прокси Dynmap. По умолчанию: `8123`. |

## Постійне Зберігання

Смонтируйте `/eaglerX-1.8-server` для сохранения миров, плагинов, конфигурации
сервера и браузерных assets. Когда смонтированная директория пуста, контейнер
инициализирует ее из образа перед запуском выбранной среды.

Рекомендуемая структура хоста:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Держите каждую версию в своей директории, так как активные пути `server/` и
`web/` являются версионными символическими ссылками, создаваемыми при запуске.

## Сборка Из Upstream

Соберите такой же образ локально из upstream-репозитория Gitee:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Запустите локально собранный образ:

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

Workflow GitHub Actions этого репозитория публикует GHCR-образ, используемый
downstream-развертываниями.

## Усунення Несправностей

### Контейнер сразу завершается

Установите `MINECRAFT_VERSION` в одно из поддерживаемых значений:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### Admin-панель не принимает команды

Запустите контейнер с `RCON_PASSWORD` и откройте порт `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Данные мира исчезают после перезапуска

Смонтируйте полную директорию runtime:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Используйте стабильный путь хоста для каждой версии сервера.

### Одновременный запуск обеих версий вызывает конфликт портов

Сопоставьте второй контейнер с другими портами хоста:

```bash
-p 5300:5200 -p 5301:5201
```

## Джерела Та Подяки

- upstream-проект сервера: [mirrorvim/eaglerX-1.8-server][upstream]
- Опубликованный образ: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Оригинальная работа Eaglercraft и EaglercraftX: lax1dude (Calder Young)
- Работа Eaglercraft server: ayunami2000

## Ліцензія

Этот репозиторий лицензирован по [MIT License][license]. Перед повторным
распространением проверьте лицензии upstream-проекта и включенных компонентов.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
