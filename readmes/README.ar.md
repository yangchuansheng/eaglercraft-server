<!-- markdownlint-disable MD013 -->

# صورة Docker لخادم EaglercraftX

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
**العربية** |
[Русский](README.ru.md) |
[Українська](README.uk.md) |
[Türkçe](README.tr.md)

<!-- README-I18N:END -->

ينشر هذا المستودع صورة Docker لخوادم EaglercraftX. تُبنى الصورة من أحدث مصدر
upstream في
[mirrorvim/eaglerX-1.8-server][upstream] وتُنشر في GitHub
Container Registry باسم
[`ghcr.io/yangchuansheng/eaglerx1.8server`][container-package].

يجمع مشروع upstream عملاء المتصفح، ووصول WebSocket عبر Bungee/Waterfall،
وتشغيل Paper في صورة واحدة. عند بدء الحاوية، يحدد `MINECRAFT_VERSION` بيئة
التشغيل النشطة.

## ما الذي توفره هذه الصورة

| العنصر | القيمة |
| --- | --- |
| الصورة | `ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1` |
| مصدر البناء | `https://gitee.com/mirrorvim/eaglerX-1.8-server` |
| إصدارات الخادم المدعومة | Paper `1.8.8` and Paper `1.12.2` |
| محدد الإصدار | `MINECRAFT_VERSION=1.8` or `MINECRAFT_VERSION=1.12` |
| مدخل اللعبة والمتصفح | Port `5200` |
| جسر admin و RCON | Port `5201`, enabled with `RCON_PASSWORD` |
| مسار الاستمرارية الموصى به | `/eaglerX-1.8-server` |

## النشر على Sealos

ينشر قالب Sealos App Store هذه الصورة كخادم ذي حالة مع وصول WebSocket، ونقطة
HTTP/admin، وتخزين دائم لعوالم Overworld وNether وThe End.

[![Deploy on Sealos][sealos-badge]][sealos-template]

1. افتح [قالب EaglerCraft Server][sealos-template].
2. انقر على "Deploy Now".
3. انتظر حتى يكتمل النشر على Sealos.
4. استخدم نقطة 5200 للعب من المتصفح ومدخل خادم اللاعبين المتعددين.
5. استخدم نقطة 5201 لواجهة HTTP/admin.

تتم صيانة مصدر القالب في
[`labring-actions/templates`][sealos-template-source].

## بدء سريع باستخدام Docker

اسحب الصورة المنشورة:

```bash
docker pull ghcr.io/yangchuansheng/eaglerx1.8server:1.12.1
```

شغّل خادم Paper 1.12.2 مع بيانات دائمة ولوحة admin:

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

شغّل خادم Paper 1.8.8:

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

افتح عميل المتصفح على:

```text
http://localhost:5200
```

افتح لوحة admin على:

```text
http://localhost:5201/admin
```

استخدم قيمة `RCON_PASSWORD` عندما تطلب لوحة admin كلمة المرور.

## سلوك وقت التشغيل

تحتوي الصورة على شجرتي الإصدار:

```text
server-1.8/   Paper 1.8.8 runtime
server-1.12/  Paper 1.12.2 runtime
web-1.8/      EaglercraftX 1.8 browser client
web-1.12/     EaglercraftX 1.12 browser client
```

عند البدء، يتحقق `script/start_server.sh` من `MINECRAFT_VERSION`، وينشئ
الروابط الرمزية النشطة، ويكتب `server/eula.txt`، ويضبط RCON. ثم يشغّل
Bungee/Waterfall وPaper ضمن `tmux`.

عند `MINECRAFT_VERSION=1.12` تصبح المسارات النشطة:

```text
web/    -> web-1.12/
server/ -> server-1.12/
```

عند `MINECRAFT_VERSION=1.8` تصبح المسارات النشطة:

```text
web/    -> web-1.8/
server/ -> server-1.8/
```

شغّل حاويات منفصلة مع أدلة مضيف منفصلة عند تقديم الإصدارين في الوقت نفسه:

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

تتوفر الحاوية الثانية على `http://localhost:5300` و
`http://localhost:5301/admin`.

## الإعدادات

| المتغير | مطلوب | الوصف |
| --- | --- | --- |
| `MINECRAFT_VERSION` | نعم | يحدد `1.8` أو `1.12`. |
| `RCON_PASSWORD` | لا | يفعل RCON وواجهة admin API على المنفذ `5201`. |
| `SERVER_DATA_DIR` | لا | مسار استمرارية قديم للعوالم فقط. |
| `ADMIN_AUTH_SECRET` | لا | يستبدل سر توقيع رمز admin. |
| `ADMIN_AUTH_TOKEN_TTL` | لا | يضبط عمر رمز admin بالثواني. |
| `DYNMAP_HOST` | لا | مضيف وكيل Dynmap اختياري. الافتراضي: `127.0.0.1`. |
| `DYNMAP_PORT` | لا | منفذ وكيل Dynmap اختياري. الافتراضي: `8123`. |

## الاستمرارية

اربط `/eaglerX-1.8-server` لحفظ العوالم والإضافات وإعدادات الخادم وأصول
المتصفح. عندما يكون الدليل المرتبط فارغًا، تهيئه الحاوية من الصورة قبل بدء
بيئة التشغيل المختارة.

تخطيط المضيف الموصى به:

```text
/data/eagler-1.8/   data for the Paper 1.8.8 container
/data/eagler-1.12/  data for the Paper 1.12.2 container
```

احتفظ بكل إصدار في دليله الخاص لأن مساري `server/` و `web/` النشطين روابط
رمزية خاصة بالإصدار يتم إنشاؤها عند البدء.

## البناء من Upstream

ابنِ نفس نوع الصورة محليًا من مستودع Gitee upstream:

```bash
git clone https://gitee.com/mirrorvim/eaglerX-1.8-server.git
cd eaglerX-1.8-server
docker build -t ghcr.io/yangchuansheng/eaglerx1.8server:local .
```

شغّل الصورة المبنية محليًا:

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

ينشر workflow الخاص بـ GitHub Actions في هذا المستودع صورة GHCR المستخدمة في
عمليات النشر downstream.

## استكشاف الأخطاء وإصلاحها

### تخرج الحاوية فورًا

اضبط `MINECRAFT_VERSION` على إحدى القيم المدعومة:

```bash
-e MINECRAFT_VERSION=1.8
-e MINECRAFT_VERSION=1.12
```

### لوحة admin لا تقبل الأوامر

ابدأ الحاوية مع `RCON_PASSWORD` وافتح المنفذ `5201`:

```bash
-p 5201:5201 -e RCON_PASSWORD=change-this-password
```

### تختفي بيانات العالم بعد إعادة التشغيل

اربط دليل وقت التشغيل الكامل:

```bash
-v /data/eagler-1.12:/eaglerX-1.8-server
```

استخدم مسار مضيف ثابتًا لكل إصدار خادم.

### تشغيل الإصدارين في الوقت نفسه يسبب تعارض منافذ

اربط الحاوية الثانية بمنافذ مضيف مختلفة:

```bash
-p 5300:5200 -p 5301:5201
```

## المصادر والاعتمادات

- مشروع الخادم upstream: [mirrorvim/eaglerX-1.8-server][upstream]
- الصورة المنشورة: [ghcr.io/yangchuansheng/eaglerx1.8server][container-package]
- عمل Eaglercraft و EaglercraftX الأصلي: lax1dude (Calder Young)
- عمل Eaglercraft server: ayunami2000

## الترخيص

هذا المستودع مرخص بموجب [MIT License][license]. راجع مشروع upstream والمكونات
المضمنة لمعرفة تراخيصها قبل إعادة التوزيع.

[container-badge]: https://img.shields.io/badge/GHCR-eaglerx1.8server-blue
[container-package]: https://github.com/yangchuansheng/eaglercraft-server/pkgs/container/eaglerx1.8server
[license]: ../LICENSE
[license-badge]: https://img.shields.io/badge/license-MIT-green
[sealos-badge]: https://sealos.io/Deploy-on-Sealos.svg
[sealos-template]: https://sealos.io/products/app-store/eaglercraft-server
[sealos-template-source]: https://github.com/labring-actions/templates/tree/kb-0.9/template/eaglercraft-server
[upstream]: https://gitee.com/mirrorvim/eaglerX-1.8-server
