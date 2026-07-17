# VibeCode Deploy Agent Note

Копируемая памятка для будущих проектов. Нужна, чтобы агент не путался в ключах VibeCode, не просил лишний ключ у пользователя и правильно доводил приложение до видимости внутри Битрикс24.

## Главное

Для деплоя и встраивания приложения в Битрикс24 обычно нужны две разные сущности:

1. `vibe_api_...` — персональный API-ключ VibeCode.
   - используется агентом для управления платформой;
   - через него можно создавать OAuth-приложения, серверы, деплоить код, смотреть статус и логи;
   - хранится в `.env` как `VIBECODE_API_KEY`;
   - не должен попадать в архив деплоя.

2. `vibe_app_...` — raw key OAuth-приложения.
   - если его ещё нет, агент должен сначала попробовать создать OAuth-приложение сам через `POST /v1/apps` с `vibe_api_...`;
   - `rawKey` из ответа `POST /v1/apps` и есть нужный `vibe_app_...`;
   - используется уже самим задеплоенным приложением;
   - используется для привязки placements;
   - нужен для чтения данных портала через VibeCode Gateway/Entity API;
   - передаётся в env сервера как `VIBE_APP_KEY`;
   - не заменяет `vibe_api_...` и не используется как ключ для infra-deploy.

Не надо просить у пользователя “ещё один персональный ключ”, если уже есть:

- `VIBECODE_API_KEY` в `.env`;
- VibeCode server, куда можно деплоить.

Если `/v1/me` с personal key говорит, что personal key не может bind placements / transparent iframe auth, это ожидаемо. Это не означает, что нужен новый personal key. Нужно создать или найти OAuth-приложение и получить его `rawKey` (`vibe_app_...`).

## Правильная модель

```text
локальный код
  │
  │ infra deploy через vibe_api_...
  ▼
VibeCode server
  │
  │ runtime env: VIBE_APP_KEY=vibe_app_...
  ▼
приложение внутри placement Битрикс24
  │
  │ VIBE_APP_KEY + X-Vibe-Authorization от VibeCode Gateway
  ▼
VibeCode Entity API / Batch
  │
  ▼
данные портала Битрикс24
```

## Правильный порядок действий

1. Прочитать `.env`.
   - Для infra API нужен `VIBECODE_API_KEY=vibe_api_...`.
   - Не печатать ключ в логах.

2. Вызвать:

```text
GET /v1/me
```

   - Если ключ personal (`vibe_api_...`) — это нормально для deploy.
   - Если `/v1/me.portalEmbedding` говорит, что personal key не может bind placements, не останавливаться и не просить новый personal key.

3. Создать OAuth-приложение, если локально ещё нет `vibe_app_...`:

```text
POST /v1/apps
X-Api-Key: <VIBECODE_API_KEY / vibe_api_...>
```

   - Использовать документацию и `/v1/me`/`/v1/guide`, чтобы взять актуальный payload.
   - В payload обычно нужны имя приложения, URL задеплоенного приложения или app/server binding, scopes/placements по документации.
   - Из ответа взять `rawKey`.
   - `rawKey` должен начинаться с `vibe_app_...`.
   - Сохранить его локально в `.env` как `VIBE_APP_KEY=...`.
   - Не коммитить `.env`, не класть его в deploy-архив.

4. Деплоить сервер через infra API всё равно с `vibe_api_...`.

5. Передать `VIBE_APP_KEY` в env задеплоенного сервера.

6. Привязать placements через VibeCode API, используя OAuth app context/key по документации.

## Что важно для видимости в портале

Публикация в каталог и видимость приложения в интерфейсе — не одно и то же.

Для MVP может быть достаточно:

1. задеплоить сервер приложения на VibeCode;
2. создать OAuth-приложение через `POST /v1/apps` или использовать уже созданное;
3. привязать placements напрямую через VibeCode API.

Типовые рабочие placements для CRM-приложения:

```text
LEFT_MENU
CRM_COMPANY_DETAIL_TAB
```

Handler для placements должен быть VibeCode handler:

```text
https://vibecode.bitrix24.tech/v1/bitrix-handler
```

Не подставляй напрямую `https://app-xxx.vibecode.bitrix24.tech` как handler placement, если документация/платформа ожидает VibeCode handler. Именно handler прокидывает gateway-авторизацию.

Сам `appUrl` задеплоенного приложения нужен OAuth-приложению/handler flow как целевой URL, но placement handler должен оставаться:

```text
https://vibecode.bitrix24.tech/v1/bitrix-handler
```

## Типовой деплой

Перед деплоем:

- собрать проект;
- создать архив без `.env`, `.git`, `node_modules`, `coverage`, `work`;
- передать `VIBE_APP_KEY` в env сервера;
- стартовать приложение командой production-start.

Примерная схема:

```bash
npm run build

tar \
  --exclude='.env' \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='coverage' \
  --exclude='work' \
  -czf /tmp/app-deploy.tgz .
```

Payload деплоя должен содержать:

```json
{
  "source": {
    "content": "<base64 архива>"
  },
  "runtime": "node20",
  "install": "cd /opt/app && npm ci --omit=dev",
  "start": "cd /opt/app && npm run start",
  "port": 3000,
  "env": {
    "NODE_ENV": "production",
    "VIBE_APP_KEY": "<raw vibe_app key>"
  }
}
```

Запрос деплоя идёт с персональным ключом:

```text
X-Api-Key: <VIBECODE_API_KEY / vibe_api_...>
```

Важно:

- `vibe_app_...` не используется как infra deploy key.
- `vibe_app_...` передаётся внутрь runtime env как `VIBE_APP_KEY`.
- Если `VIBE_APP_KEY` ещё нет, сначала создать OAuth-приложение через `POST /v1/apps` с `vibe_api_...`.

## После деплоя проверить

1. Статус сервера:

```text
GET /v1/infra/servers/:serverId
```

Ожидаемо:

```json
{
  "status": "running",
  "blackholeStatus": "CONNECTED"
}
```

2. Логи сервера:

```text
GET /v1/infra/servers/:serverId/logs?lines=60
```

Для Node-приложения ожидаемо что-то вроде:

```text
npm run start
node dist/...
... listening on http://0.0.0.0:3000
```

3. Внешний smoke-test:

```text
POST /v1/infra/servers/:serverId/access-tokens { "mode": "api-bearer" }
GET <appUrl>/api/health
Authorization: Bearer <returned token>
```

Не сохранять preview token в репозиторий. Если он был записан во временный файл, удалить после проверки.

## Auto-sleep

Если сервер “не запускается”, сначала проверь статус.

Частый случай:

```json
{
  "status": "sleeping",
  "blackholeStatus": "DISCONNECTED"
}
```

Это не ошибка кода. Сервер уснул из-за `sleepAfterMinutes`.

Решения:

- разбудить повторным деплоем;
- увеличить auto-sleep, например до `240`;
- отключить sleep, если допустим постоянный расход баланса.

## Типичные ошибки агента

1. Ошибка: думать, что для деплоя нужен `vibe_app_...`.
   - правильно: infra-deploy делается через `vibe_api_...`;
   - `vibe_app_...` нужен runtime-приложению.

2. Ошибка: если `/v1/me` говорит, что personal key не может bind placements, просить у пользователя новый ключ.
   - правильно: это ожидаемо для `vibe_api_...`;
   - сначала создать OAuth-приложение через `POST /v1/apps`;
   - взять `rawKey` из ответа;
   - сохранить его как `VIBE_APP_KEY`;
   - только потом работать с placements.

3. Ошибка: думать, что нужен новый personal API-ключ.
   - правильно: сначала проверить `.env`, существующие app/server ids, `/v1/me`, `/v1/apps`, логи и статус сервера.

4. Ошибка: пытаться публиковать в каталог, если задача — просто сделать приложение видимым.
   - правильно: для MVP достаточно placements bind.

5. Ошибка: деплоить `.env` или `node_modules`.
   - правильно: `.env` исключить, зависимости ставить на сервере через `npm ci --omit=dev`.

6. Ошибка: считать sleeping server падением приложения.
   - правильно: проверить `status`, `blackholeStatus`, логи и auto-sleep.

7. Ошибка: оставлять `work/*.json` с ответами API в deploy-архиве.
   - правильно: исключить `work/`; особенно не хранить preview access tokens.

## Короткое сообщение для агента

Не нужен новый персональный ключ. Используй `VIBECODE_API_KEY` (`vibe_api_...`) из `.env` для VibeCode infra API: create app/server, deploy/status/logs. Если `/v1/me` говорит, что personal key не может bind placements / transparent iframe auth, это ожидаемо: для этого нужен `vibe_app_...`, то есть raw key OAuth-приложения. Не проси его сразу у пользователя: сначала создай OAuth-приложение через `POST /v1/apps`, возьми `rawKey` из ответа и сохрани локально как `VIBE_APP_KEY`. `vibe_app_...` передаётся в env сервера как `VIBE_APP_KEY` и используется для placements bind; им не деплоят infra. Архив деплоя собирай без `.env`, `.git`, `node_modules`, `coverage`, `work`; runtime `node20`, install `npm ci --omit=dev`, start `npm run start`, port `3000`. Для видимости в Битрикс24 можно привязать placements `LEFT_MENU` и `CRM_COMPANY_DETAIL_TAB` через VibeCode API, handler — `https://vibecode.bitrix24.tech/v1/bitrix-handler`. Если сервер не открывается, сначала проверь `status`: `sleeping` означает auto-sleep, а не падение приложения.

---

## Дополнение: проверено на практике

Раздел добавлен к исходной памятке. Всё ниже подтверждено вызовами API, а не вычитано из документации.

### bind работает без согласия пользователя, publish — нет

Это главное. Два разных пути, и выбор неверного упирает в тупик:

```text
POST /v1/placements/bind      X-Api-Key: vibe_app_...     -> работает сразу
POST /v1/apps/:id/publish     -> 400 NO_USER_TOKEN
```

`bind` идёт dev-key путём и требует только ключ приложения. `publish` привязывает встройки OAuth-токеном пользователя, которого у свежего приложения нет, — и получить его можно лишь через страницу согласия в браузере. Публикация нужна только для каталога; для видимости встройки — нет.

**Документация здесь врёт.** В `/v1/me` поле `api._rules` утверждает: «With an OAuth app key: POST /v1/placements/bind (needs placement scope + Bearer session)». Bearer session не нужен. При расхождении доков и практики — пробуй эндпоинт.

### Токен приложения создаёт сам обработчик

У свежего приложения `/v1/me` с его ключом показывает `currentUser: null`. Это не проблема: обработчик выпускает OAuth-токен при первом открытии встройки сотрудником. Схема самозагружается — сначала bind, дальше само.

Отсюда же следствие: новый `vibe_app_` ключ из кабинета ничем не поможет, он будет в том же состоянии. Дело не в ключе.

### publish дополнительно требует снапшот

Если publish всё же нужен, перед ним обязателен свежий снапшот исходников, иначе `409 SNAPSHOT_REQUIRED`:

```text
POST /v1/apps/:id/sources     Content-Type: application/gzip
```

Снапшот сервера, который деплой сохраняет сам, не считается — нужен снапшот приложения.

### Мелочи, стоившие времени

- `tar` на macOS кладёт в архив `._*` и `.DS_Store`. Собирай с `COPYFILE_DISABLE=1`.
- `curl` трактует `[` и `]` в URL как glob — фильтры вида `filter[companyId]=4` уходят пустыми. Нужен флаг `-g` / `--globoff`.
- `accessPolicy` по умолчанию `OWNER_ONLY`: шлюз отдаёт свою заглушку вместо приложения. Для приложения за платформенным обработчиком правильное значение — `PORTAL`.
- Создание сервера принимает `Idempotency-Key` — повтор после потерянного ответа не заведёт вторую платную машину.
