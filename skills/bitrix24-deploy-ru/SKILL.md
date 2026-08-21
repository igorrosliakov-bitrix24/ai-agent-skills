---
name: bitrix24-deploy-ru
description: "Помогает на русском деплоить приложения для Битрикс24 через VibeCode: различать vibe_api и vibe_app ключи, собирать архив без секретов, передавать VIBE_APP_KEY, bind placements и проверять статус, логи и auto-sleep. Используй первым для русскоязычных задач про деплой, публикацию, размещение, bind placements, VibeCode server или видимость приложения в Битрикс24."
---

# Деплой приложений Битрикс24

Используй этот навык первым, когда пользователь просит задеплоить приложение для Битрикс24, сделать его видимым в портале, привязать placements, проверить VibeCode server, разобраться с ключами VibeCode или довести приложение до рабочего размещения.

## Главное

- Не проси новый ключ, пока не проверил существующие `.env`, app/server ids, временные ответы создания OAuth-приложения и возможность создать OAuth-приложение через API.
- Различай ключи: `vibe_api_...` управляет VibeCode infra API, а `vibe_app_...` работает в runtime как `VIBE_APP_KEY` и нужен для Gateway/Entity API и bind placements.
- Для видимости MVP в портале часто достаточно deploy server + OAuth app + placements bind; publish в каталог не обязателен.

## Порядок работы

1. Перед любыми действиями прочитай `references/vibecode-deploy.md`.
2. Найди локальные значения: `VIBECODE_API_KEY`, `VIBE_APP_KEY`, `VIBECODE_APP_ID`, server id, app URL и прошлые `/tmp/*app-create*.json`.
3. Если `VIBE_APP_KEY` отсутствует, сначала попробуй создать OAuth-приложение через `POST /v1/apps` с `vibe_api_...` и взять `rawKey`.
4. Собирай deploy-архив без `.env`, `.git`, `node_modules`, `coverage`, `work` и любых preview/access tokens.
5. В deploy payload передавай runtime env `VIBE_APP_KEY=<raw vibe_app key>`; infra-запросы делай с `VIBECODE_API_KEY`.
6. Для placements используй raw app key `vibe_app_...` и handler `https://vibecode.bitrix24.tech/v1/bitrix-handler`.
7. После деплоя проверь server status, `blackholeStatus`, app URL и логи; `sleeping` трактуй как auto-sleep, а не как падение приложения.

## Обязательные проверки

- Не отправляй `.env` и секреты в архив деплоя.
- Не используй `vibe_app_...` как infra deploy key.
- Не используй `vibe_api_...` для bind placements, если endpoint требует OAuth app key.
- Не подставляй `https://app-xxx.vibecode.bitrix24.tech` как placement handler, когда нужен VibeCode handler.
- Если платформа возвращает, что personal key не умеет bind placements или transparent iframe auth, это ожидаемо: нужен `vibe_app_...`, а не новый personal key.

## Материалы

- Памятка по VibeCode deploy: `references/vibecode-deploy.md`
