# DEPLOY — как публиковать и обновлять воронку

Единый источник правды по деплою. Любой чат (или человек) читает этот файл и делает всё сам — знание не живёт в памяти чата.

## Что это
- **Репозиторий:** `anna-holubeva/facestudio-funnel` (public)
- **Хостинг:** GitHub Pages (ветка `main`, корень `/`), сеть Fastly — открывается и в РФ, и в Украине
- **Домен:** `go.facestudio.online` — ✅ подключён (SSL, Enforce HTTPS, проверен РФ+Украина)
- **Живые страницы:** https://go.facestudio.online/diagnostika/ · /video1/ · /rezultaty/ · /trenirovka/ · /bonus/ · /rules/
- github.io-адреса теперь 301-редиректят на домен — это нормально
- **Локальный клон:** `D:\Обучение\Boost\Работа с FS\facestudio-funnel\`
- **Токен:** `GITHUB_TOKEN_FACESTUDIO` в `D:\Обучение\Boost\Работа с FS\EN\secrets.local.env` (scope repo). В репозиторий НЕ коммитить — он публичный.

## Структура
```
/                     index.html   — заглушка-корень
/diagnostika/         index.html   — лендинг видео-диагностики (вход воронки)
/video1/              index.html   — страница видео-диагностики (шаг 3: плеер + опрос)
/rezultaty/           index.html   — страница результатов диагностики (разбор + план + подарок)
/trenirovka/          index.html   — бесплатная тренировка (шаг 7: плеер + 72ч-доступ, 2 состояния)
/bonus/               index.html   — оффер (шаг 8: вступление в Клуб — тарифы, бонусы 24ч, кнопки в бота)
/rules/               index.html   — противопоказания к тренировкам (перенос с facestudio.by, открывается в Украине)
.nojekyll                          — отключает Jekyll (статика как есть)
CNAME                              — go.facestudio.online (домен подключён)
```

## Как обновить страницу (деплой = git push)
1. Правишь нужный `index.html` (например, `diagnostika/index.html`).
2. Из Git Bash:
```bash
REPO="/d/Обучение/Boost/Работа с FS/facestudio-funnel"
SF="/d/Обучение/Boost/Работа с FS/EN/secrets.local.env"
TOKEN=$(grep -E '^GITHUB_TOKEN_FACESTUDIO=' "$SF" | cut -d= -f2- | tr -d '"'\''\r ')
git -C "$REPO" add -A
git -C "$REPO" commit -m "update diagnostika"
git -C "$REPO" push "https://x-access-token:$TOKEN@github.com/anna-holubeva/facestudio-funnel.git" main
```
3. GitHub Pages пересобирается за ~30–60 сек. Адрес не меняется. История деплоев — `git log`.

> Лендинги — самодостаточные HTML (шрифты/картинки/CSS/JS вшиты base64). Просто заменить файл — этого достаточно.

## Подключение домена go.facestudio.online (одноразово) — ✅ УЖЕ СДЕЛАНО 12.07.2026
> Домен подключён и работает. Раздел ниже — на случай пересоздания/отладки.
> Нюанс: custom domain задан через Pages API (`PUT /repos/.../pages {"cname":"go.facestudio.online"}`) — файл `CNAME` сам не всегда подхватывается.

DNS-зоной `facestudio.online` управляет **Cloudflare** (не reg.ru). Записи добавляет чат, ведущий DNS.
1. **Cloudflare → DNS →** добавить запись:
   - Type: `CNAME` · Name: `go` · Target: `anna-holubeva.github.io` · Proxy: **DNS only (серая тучка)** · TTL: Auto
2. В этом репозитории создать файл `CNAME` (без расширения) с одной строкой:
   ```
   go.facestudio.online
   ```
   закоммитить и запушить. GitHub Pages подхватит домен и выпустит SSL (Let's Encrypt) за несколько минут.
3. В Settings → Pages репозитория убедиться, что Custom domain = `go.facestudio.online` и включён «Enforce HTTPS».

## Проверка доступности по странам (обязательно перед запуском трафика)
check-host.net, узлы РФ + Украина:
```
https://check-host.net/check-http?host=https://go.facestudio.online/diagnostika/&node=ru1.node.check-host.net&node=ru2.node.check-host.net&node=ua1.node.check-host.net&node=ua2.node.check-host.net
```
затем poll `https://check-host.net/check-result/<request_id>`. Должно быть 200 со всех узлов.

## Правила
- Репозиторий **публичный** → никаких токенов/секретов внутрь (см. `.gitignore`).
- Прокси Cloudflare для этого домена НЕ включать (ломает в РФ и мешает SSL GitHub).
- Видео воронки — на **Kinescope** (работает в РФ и Украине).
