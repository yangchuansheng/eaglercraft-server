<!-- markdownlint-disable MD013 -->

# EaglercraftX Server Docker イメージ

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
**日本語** |
[العربية](README.ar.md) |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

このリポジトリは EaglercraftX サーバー用の Docker イメージを公開します。 このイメージは
[mirrorvim/eaglerX-1.8-server][upstream] の最新 upstream ソースから ビルドされ、GitHub
Container Registry に
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package] として公開されます。

upstream プロジェクトは、ブラウザクライアント、Bungee/Waterfall WebSocket アクセス、Paper ランタイムを 1
つのイメージにまとめています。コンテナ起動時に `MINECRAFT_VERSION` が有効なランタイムを選択します。

## このイメージが提供するもの

| 項目 | 値 |
| --- | --- |
| イメージ | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| ビルド元 | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| 対応サーバーバージョン | Paper `1.8.8` and Paper `1.12.2` |
| バージョン選択 | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| ゲームおよびブラウザ入口 | Port `5200` |
| Admin および RCON ブリッジ | Port `5201`, enabled with `RCON_PASSWORD` |
| 推奨永続化パス | `/eaglerX-1.8-server` |

## Sealos にデプロイ

Sealos App Store テンプレートは、このイメージを WebSocket アクセス、 HTTP/admin
エンドポイント、Overworld、Nether、The End 用の永続ストレージを備えた ステートフルサーバーとしてデプロイします。

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. [EaglerCraft Server テンプレート][sealos-template]を開きます。
2. "Deploy Now" をクリックします。
3. Sealos のデプロイが完了するまで待ちます。
4. ブラウザプレイとマルチプレイヤーサーバー入口には 5200 エンドポイントを使います。
5. HTTP/admin 画面には 5201 エンドポイントを使います。

テンプレートソースは
[`labring-actions/templates`][sealos-template-source] で管理されています。

## Docker クイックスタート

公開済みイメージを取得します:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

永続データと admin パネル付きの Paper 1.12.2 サーバーを実行します:

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

Paper 1.8.8 サーバーを実行します:

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

ブラウザクライアントを開きます:

```text
http://localhost:5200
```

admin パネルを開きます:

```text
http://localhost:5201/admin
```

admin パネルがパスワードを求めたら `RCON_PASSWORD` の値を使います。

## ランタイム動作

イメージには両方のバージョンツリーが含まれています:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

起動時に `script/start_server.sh` は `MINECRAFT_VERSION` を検証し、アクティブな
シンボリックリンクを作成し、`server/eula.txt` を書き込み、RCON を設定します。その後 `tmux` 配下で
Bungee/Waterfall と Paper を起動します。

`MINECRAFT_VERSION=1.12` の場合、有効なパスは次のようになります:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

`MINECRAFT_VERSION=1.8` の場合、有効なパスは次のようになります:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

両方のバージョンを同時に提供する場合は、別々のホストディレクトリで 別々のコンテナを実行します:

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

2 つ目のコンテナは `http://localhost:5300` と `http://localhost:5301/admin` で利用できます。

## 設定

| 変数 | 必須 | 説明 |
| --- | --- | --- |
| `MINECRAFT_VERSION` | はい | `1.8` または `1.12` を選択します。 |
| `RCON_PASSWORD` | いいえ | `5201` ポートで RCON と admin API を有効にします。 |
| `SERVER_DATA_DIR` | いいえ | 従来の world 専用永続化パスです。 |
| `ADMIN_AUTH_SECRET` | いいえ | admin トークン署名シークレットを上書きします。 |
| `ADMIN_AUTH_TOKEN_TTL` | いいえ | admin トークンの有効期間を秒単位で設定します。 |
| `DYNMAP_HOST` | いいえ | 任意の Dynmap プロキシホスト。既定: `127.0.0.1`。 |
| `DYNMAP_PORT` | いいえ | 任意の Dynmap プロキシポート。既定: `8123`。 |

## 永続化

world、plugin、サーバー設定、ブラウザ asset を保持するには `/eaglerX-1.8-server` を
マウントします。マウント先ディレクトリが空の場合、コンテナは選択されたランタイムを 開始する前にイメージから初期化します。

推奨ホスト構成:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

有効な `server/` と `web/` パスは起動時に作成されるバージョン固有の
シンボリックリンクなので、各バージョンを個別のディレクトリに保持します。

## Upstream からビルド

upstream Gitee リポジトリから同じ形式のイメージをローカルでビルドします:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

ローカルでビルドしたイメージを実行します:

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

このリポジトリの GitHub Actions workflow は、downstream デプロイで使われる GHCR イメージを公開します。

## トラブルシューティング

### コンテナがすぐ終了する

`MINECRAFT_VERSION` を対応値のいずれかに設定します:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### admin パネルがコマンドを受け付けない

`RCON_PASSWORD` でコンテナを起動し、`5201` ポートを公開します:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### 再起動後に world データが消える

完全なランタイムディレクトリをマウントします:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

各サーバーバージョンに安定したホストパスを使います。

### 両方のバージョンを同時に実行するとポートが競合する

2 つ目のコンテナを別のホストポートにマッピングします:

```bash
-p 5300:5200 -p 5301:5201
```

## ソースとクレジット

- upstream サーバープロジェクト: [mirrorvim/eaglerX-1.8-server][upstream]
- 公開済みイメージ: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- Eaglercraft と EaglercraftX の原作: lax1dude (Calder Young)
- Eaglercraft server の作業: ayunami2000

## ライセンス

このリポジトリは [MIT License][license] の下でライセンスされています。再配布の前に、 upstream
プロジェクトと同梱コンポーネントそれぞれのライセンスを確認してください。

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
