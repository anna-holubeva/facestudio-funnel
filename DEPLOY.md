# DEPLOY — как публиковать и обновлять воронку

Единый источник правды по деплою. Любой чат (или человек) читает этот файл и делает всё сам — знание не живёт в памяти чата.

## Что это
- **Репозиторий:** `anna-holubeva/facestudio-funnel` (public)
- **Хостинг:** GitHub Pages (ветка `main`, корень `/`), сеть Fastly — открывается и в РФ, и в Украине
- **Домен:** `go.facestudio.online` (после настройки DNS, см. ниже)
- **Превью до DNS:** https://anna-holubeva.github.io/facestudio-funnel/
- **Локальный клон:** `D:\Обучение\Boost\Работа с FS\facestudio-funnel\`
- **Токен:** `GITHUB_TOKEN_FACESTUDIO` в `D:\Обучение\Boost\Работа с FS\EN\secrets.local.env` (scope repo). В репозиторий НЕ коммитить — он публичный.

## Структура
```
/                     index.html   — заглушка-корень
/diagnostika/         index.html   — лендинг видео-диагностики (вход воронки)
.nojekyll                          — отключает Jekyll (статика как есть)
CNAME                              — добавить ПОСЛЕ настройки DNS (см. ниже)
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

## Подключение домена go.facestudio.online (одноразово)
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
