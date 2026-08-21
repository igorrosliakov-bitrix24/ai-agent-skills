# VibeCode deploy для Битрикс24

Короткая памятка для задач, где агент путается в ключах VibeCode, деплое сервера, runtime env и видимости приложения внутри Битрикс24.

## Ключевая модель

Для деплоя и встраивания приложения в Битрикс24 обычно нужны две разные сущности:

1. `vibe_api_...` — персональный API-ключ VibeCode.
   - Используется агентом для управления платформой.
   - Через него можно создавать и читать серверы, деплоить код, смотреть статус и логи.
   - Хранится в `.env` как `VIBECODE_API_KEY`.
   - Не должен попадать в архив деплоя.

2. `vibe_app_...` — raw key OAuth-приложения.
   - Используется уже задеплоенным приложением.
   - Нужен для чтения данных портала через VibeCode Gateway/Entity API.
   - Передаётся в env сервера как `VIBE_APP_KEY`.
   - Не заменяет `vibe_api_...` и не используется как ключ для infra-deploy.

Схема:

```text
локальный код
  |
  | деплой через vibe_api_...
  v
VibeCode server
  |
  | runtime env: VIBE_APP_KEY=vibe_app_...
  v
приложение внутри placement Битрикс24
  |
  | VIBE_APP_KEY + X-Vibe-Authorization от VibeCode Gateway
  v
VibeCode Entity API / Batch
  |
  v
данные портала Битрикс24
```

Главная развилка:

```text
deploy/status/logs/servers/apps creation  -> vibe_api_...
runtime чтение данных портала             -> vibe_app_... + X-Vibe-Authorization
bind placements / transparent iframe auth -> vibe_app_...
```

## Не просить лишний ключ

Не создавай и не проси ещё один ключ, если уже есть:

- `VIBECODE_API_KEY` в `.env`;
- созданное OAuth-приложение с raw key `vibe_app_...`;
- VibeCode server, куда можно деплоить.

Если `vibe_app_...` не лежит в `.env`, это не значит, что пользователь должен вручную создать новый ключ. Сначала проверь:

- не создавалось ли OAuth-приложение ранее через `POST /v1/apps`;
- нет ли сохранённого ответа создания приложения во временном файле, например `/tmp/*app-create*.json`;
- нет ли `VIBECODE_APP_ID`, `VIBE_APP_KEY` или похожих переменных в `.env`, `.env.example`, документации проекта;
- можно ли создать OAuth-приложение через API самостоятельно.

Если `/v1/me` с текущим ключом говорит, что это `personal key` и он не может `bind placements` или `transparent iframe auth`, это нормально. Это не значит “нужен ещё один персональный ключ”. Для этого шага используй raw key OAuth-приложения `vibe_app_...`. Если его нет, создай OAuth-приложение через `POST /v1/apps` и сохрани `rawKey` из ответа.

## Видимость в портале

Публикация в каталог и видимость приложения в интерфейсе — разные вещи.

Для MVP может быть достаточно:

1. Задеплоить сервер приложения на VibeCode.
2. Создать или использовать OAuth-приложение.
3. Привязать placements напрямую через VibeCode API.

Рабочие placements из прошлого проекта:

```text
LEFT_MENU
CRM_COMPANY_DETAIL_TAB
```

Handler для placements должен быть VibeCode handler:

```text
https://vibecode.bitrix24.tech/v1/bitrix-handler
```

Не подставляй напрямую `https://app-xxx.vibecode.bitrix24.tech` как handler placement, если документация или платформа ожидает VibeCode handler. Именно handler прокидывает gateway-авторизацию.

## Типовой деплой

Перед деплоем:

- Собери проект.
- Создай архив без `.env`, `.git`, `node_modules`, `coverage`, `work`.
- Передай `VIBE_APP_KEY` в env сервера.
- Запусти production-start команду.

Примерная схема:

```bash
npm run build

tar \
  --exclude='.env' \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='coverage' \
  -czf /tmp/app-deploy.tgz .
```

Payload деплоя:

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

## После деплоя

Проверь статус сервера:

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

Проверь логи:

```text
GET /v1/infra/servers/:serverId/logs?lines=60
```

Для Node-приложения ожидаемо:

```text
npm run start
node dist/server.js
<app name> is listening on http://localhost:3000
```

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

Возможные решения:

- Разбудить повторным деплоем.
- Увеличить auto-sleep, например до `240`.
- Отключить sleep, если допустим постоянный расход баланса.

## Типичные ошибки агента

1. Думать, что для деплоя нужен `vibe_app_...`.
   - Правильно: infra-deploy делается через `vibe_api_...`.
   - `vibe_app_...` нужен runtime-приложению.

2. Думать, что нужен новый API-ключ.
   - Правильно: сначала проверить `.env`, существующие app/server ids и логи.

3. Пытаться публиковать в каталог, если задача — просто сделать приложение видимым.
   - Правильно: для MVP достаточно placements bind.

4. Деплоить `.env` или `node_modules`.
   - Правильно: `.env` исключить, зависимости ставить на сервере через `npm ci --omit=dev`.

5. Считать sleeping server падением приложения.
   - Правильно: проверить `status`, `blackholeStatus`, логи и auto-sleep.

6. Увидеть в `/v1/me`, что `vibe_api_...` является personal key, и остановиться.
   - Правильно: personal key нужен для infra API.
   - Для placements нужен `vibe_app_...`.
   - `vibe_app_...` можно получить при создании OAuth-приложения через API.

## Короткое напоминание

Не нужен новый персональный ключ. Используй `VIBECODE_API_KEY` (`vibe_api_...`) из `.env` для VibeCode infra API: create app/server, deploy/status/logs. Если `/v1/me` говорит, что personal key не может bind placements / transparent iframe auth, это ожидаемо: для этого нужен `vibe_app_...`, то есть raw key OAuth-приложения. Сначала попробуй получить его сам: создай OAuth-приложение через `POST /v1/apps`, возьми `rawKey` из ответа и сохрани локально. `vibe_app_...` передаётся в env сервера как `VIBE_APP_KEY` и используется для placements bind; им не деплоят infra. Архив деплоя собирай без `.env`, `.git`, `node_modules`, `coverage`, `work`; runtime `node20`, install `npm ci --omit=dev`, start `npm run start`, port `3000`. Для видимости в Битрикс24 не обязательно publish в каталог: можно привязать placements `LEFT_MENU` и `CRM_COMPANY_DETAIL_TAB` через VibeCode API, handler — `https://vibecode.bitrix24.tech/v1/bitrix-handler`. Если сервер не открывается, сначала проверь `status`: `sleeping` означает auto-sleep, а не падение приложения.
