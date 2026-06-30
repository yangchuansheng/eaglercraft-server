<!-- markdownlint-disable MD013 -->

# EaglercraftX Server Docker İmajı

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
[Українська](README.uk.md) |
**Türkçe**

<!-- README-I18N:END -->

Bu depo EaglercraftX sunucuları için bir Docker imajı yayımlar. İmaj,
[mirrorvim/eaglerX-1.8-server][upstream] üzerindeki en güncel upstream
kaynaktan oluşturulur ve GitHub Container Registry üzerinde
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package] olarak yayımlanır.

upstream proje tarayıcı istemcilerini, Bungee/Waterfall WebSocket erişimini ve
Paper çalışma ortamlarını tek bir imajda paketler. Konteyner başlarken
`MINECRAFT_VERSION` etkin çalışma ortamını seçer.

## Bu İmaj Neler Sağlar

| Öğe | Değer |
| --- | --- |
| İmaj | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| Derleme kaynağı | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| Desteklenen sunucu sürümleri | Paper `1.8.8` and Paper `1.12.2` |
| Sürüm seçici | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| Oyun ve tarayıcı girişi | Port `5200` |
| Admin ve RCON köprüsü | Port `5201`, enabled with `RCON_PASSWORD` |
| Önerilen kalıcılık yolu | `/eaglerX-1.8-server` |

## Sealos Üzerinde Dağıt

Sealos App Store şablonu bu imajı WebSocket erişimi, HTTP/admin endpoint’i ve
Overworld, Nether, The End için kalıcı depolama içeren durumlu bir sunucu
olarak dağıtır.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. [EaglerCraft Server şablonunu][sealos-template] açın.
2. "Deploy Now" düğmesine tıklayın.
3. Sealos dağıtımının tamamlanmasını bekleyin.
4. Tarayıcı oyunu ve multiplayer sunucu girişi için 5200 endpoint’ini kullanın.
5. HTTP/admin yüzeyi için 5201 endpoint’ini kullanın.

Şablon kaynağı
[`labring-actions/templates`][sealos-template-source] içinde tutulur.

## Docker Hızlı Başlangıç

Yayımlanan imajı çekin:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

Kalıcı veri ve admin paneliyle Paper 1.12.2 sunucusu çalıştırın:

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

Paper 1.8.8 sunucusu çalıştırın:

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

Tarayıcı istemcisini açın:

```text
http://localhost:5200
```

admin panelini açın:

```text
http://localhost:5201/admin
```

admin paneli parola istediğinde `RCON_PASSWORD` değerini kullanın.

## Çalışma Zamanı Davranışı

İmaj iki sürüm ağacını da içerir:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

Başlangıçta `script/start_server.sh`, `MINECRAFT_VERSION` değerini doğrular,
etkin sembolik bağlantıları oluşturur, `server/eula.txt` yazar ve RCON
yapılandırır. Ardından `tmux` altında Bungee/Waterfall ve Paper başlatır.

`MINECRAFT_VERSION=1.12` için etkin yollar şunlardır:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

`MINECRAFT_VERSION=1.8` için etkin yollar şunlardır:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

İki sürümü aynı anda sunarken ayrı host dizinleriyle ayrı konteynerler
çalıştırın:

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

İkinci konteyner `http://localhost:5300` ve `http://localhost:5301/admin`
adreslerinde kullanılabilir.

## Yapılandırma

| Değişken | Gerekli | Açıklama |
| --- | --- | --- |
| `MINECRAFT_VERSION` | Evet | `1.8` veya `1.12` seçer. |
| `RCON_PASSWORD` | Hayır | `5201` portunda RCON ve admin API etkinleştirir. |
| `SERVER_DATA_DIR` | Hayır | Yalnızca dünyalar için eski kalıcılık yolu. |
| `ADMIN_AUTH_SECRET` | Hayır | admin token imzalama sırrını geçersiz kılar. |
| `ADMIN_AUTH_TOKEN_TTL` | Hayır | admin token ömrünü saniye cinsinden ayarlar. |
| `DYNMAP_HOST` | Hayır | İsteğe bağlı Dynmap proxy host’u. Varsayılan: `127.0.0.1`. |
| `DYNMAP_PORT` | Hayır | İsteğe bağlı Dynmap proxy port’u. Varsayılan: `8123`. |

## Kalıcılık

Dünyaları, plugin’leri, sunucu yapılandırmasını ve tarayıcı asset’lerini
korumak için `/eaglerX-1.8-server` bağlayın. Bağlanan dizin boşsa konteyner,
seçilen çalışma zamanını başlatmadan önce dizini imajdan ilklendirir.

Önerilen host düzeni:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

Etkin `server/` ve `web/` yolları başlangıçta oluşturulan sürüme özel sembolik
bağlantılar olduğu için her sürümü kendi dizininde tutun.

## Upstream Kaynaktan Derle

Aynı tür imajı upstream Gitee deposundan yerel olarak derleyin:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

Yerel olarak derlenen imajı çalıştırın:

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

Bu deponun GitHub Actions workflow’u downstream dağıtımlarda kullanılan GHCR
imajını yayımlar.

## Sorun Giderme

### Konteyner hemen çıkıyor

`MINECRAFT_VERSION` değerini desteklenen değerlerden birine ayarlayın:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### admin paneli komut kabul etmiyor

`RCON_PASSWORD` ile konteyneri başlatın ve `5201` portunu açın:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### Yeniden başlatmadan sonra dünya verileri kayboluyor

Tam çalışma zamanı dizinini bağlayın:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

Her sunucu sürümü için kararlı bir host yolu kullanın.

### İki sürümü aynı anda çalıştırmak port çakışmasına yol açar

İkinci konteyneri farklı host portlarına eşleyin:

```bash
-p 5300:5200 -p 5301:5201
```

## Kaynaklar Ve Katkılar

- upstream sunucu projesi: [mirrorvim/eaglerX-1.8-server][upstream]
- Yayımlanan imaj: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Eaglercraft ve EaglercraftX özgün çalışması: lax1dude (Calder Young)
- Eaglercraft server çalışması: ayunami2000

## Lisans

Bu depo [MIT License][license] ile lisanslanmıştır. Yeniden dağıtmadan önce
upstream projeyi ve paketlenen bileşenleri kendi lisansları açısından
inceleyin.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
