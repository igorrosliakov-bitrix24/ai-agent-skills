# VibeCode Deploy Agent Note

A copyable note for future projects. It exists so the agent does not confuse VibeCode keys, does not ask the user for an unnecessary key, and correctly brings an app to visibility inside Bitrix24.

## Key Points

Deploying and embedding an app in Bitrix24 usually needs two different things:

1. `vibe_api_...` — a personal VibeCode API key.
   - used by the agent to manage the platform;
   - creates OAuth apps and servers, deploys code, reads status and logs;
   - stored in `.env` as `VIBECODE_API_KEY`;
   - must not end up in the deploy archive.

2. `vibe_app_...` — the raw key of the OAuth app.
   - if it does not exist yet, the agent should first try to create the OAuth app itself via `POST /v1/apps` with `vibe_api_...`;
   - `rawKey` from the `POST /v1/apps` response is that `vibe_app_...`;
   - used by the deployed app itself;
   - used to bind placements;
   - needed to read portal data through the VibeCode Gateway / Entity API;
   - passed to the server env as `VIBE_APP_KEY`;
   - does not replace `vibe_api_...` and is not used as the infra-deploy key.

Do not ask the user for "another personal key" if you already have:

- `VIBECODE_API_KEY` in `.env`;
- a VibeCode server to deploy to.

If `/v1/me` with the personal key says it cannot bind placements / do transparent iframe auth, that is expected. It does not mean a new personal key is needed. Create or find an OAuth app and take its `rawKey` (`vibe_app_...`).

## The Correct Model

```text
local code
  │
  │ infra deploy via vibe_api_...
  ▼
VibeCode server
  │
  │ runtime env: VIBE_APP_KEY=vibe_app_...
  ▼
app inside a Bitrix24 placement
  │
  │ VIBE_APP_KEY + X-Vibe-Authorization from the VibeCode Gateway
  ▼
VibeCode Entity API / Batch
  │
  ▼
Bitrix24 portal data
```

## The Correct Order

1. Read `.env`.
   - The infra API needs `VIBECODE_API_KEY=vibe_api_...`.
   - Do not print the key in logs.

2. Call:

```text
GET /v1/me
```

   - A personal key (`vibe_api_...`) is fine for deploy.
   - If `/v1/me.portalEmbedding` says the personal key cannot bind placements, do not stop and do not ask for a new personal key.

3. Create the OAuth app if no `vibe_app_...` exists locally:

```text
POST /v1/apps
X-Api-Key: <VIBECODE_API_KEY / vibe_api_...>
```

   - Use the docs and `/v1/me` / `/v1/guide` to build a current payload.
   - The payload usually needs the app name, the deployed app URL or app/server binding, and scopes/placements per the docs.
   - Take `rawKey` from the response; it must start with `vibe_app_...`.
   - Save it locally in `.env` as `VIBE_APP_KEY=...`.
   - Do not commit `.env` and do not put it in the deploy archive.

4. Deploy the server through the infra API still with `vibe_api_...`.

5. Pass `VIBE_APP_KEY` into the deployed server's env.

6. Bind placements through the VibeCode API using the OAuth app context/key per the docs.

## What Matters for Portal Visibility

Publishing to the catalog and being visible in the interface are not the same thing.

For an MVP it can be enough to:

1. deploy the app server on VibeCode;
2. create an OAuth app via `POST /v1/apps` or use an existing one;
3. bind placements directly through the VibeCode API.

Typical working placements for a CRM app:

```text
LEFT_MENU
CRM_COMPANY_DETAIL_TAB
```

The placement handler must be the VibeCode handler:

```text
https://vibecode.bitrix24.tech/v1/bitrix-handler
```

Do not put `https://app-xxx.vibecode.bitrix24.tech` directly as the placement handler when the platform expects the VibeCode handler. The handler is exactly what carries Gateway authorization.

The deployed app's own `appUrl` is needed by the OAuth app / handler flow as the target URL, but the placement handler must stay:

```text
https://vibecode.bitrix24.tech/v1/bitrix-handler
```

## A Typical Deploy

Before deploying:

- build the project;
- create an archive without `.env`, `.git`, `node_modules`, `coverage`, `work`;
- pass `VIBE_APP_KEY` into the server env;
- start the app with the production-start command.

Rough scheme:

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

The deploy payload should contain:

```json
{
  "source": {
    "content": "<base64 archive>"
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

The deploy request goes with the personal key:

```text
X-Api-Key: <VIBECODE_API_KEY / vibe_api_...>
```

Important:

- `vibe_app_...` is not used as the infra deploy key.
- `vibe_app_...` is passed into the runtime env as `VIBE_APP_KEY`.
- If `VIBE_APP_KEY` does not exist yet, first create the OAuth app via `POST /v1/apps` with `vibe_api_...`.

## Check After Deploy

1. Server status:

```text
GET /v1/infra/servers/:serverId
```

Expected:

```json
{
  "status": "running",
  "blackholeStatus": "CONNECTED"
}
```

2. Server logs:

```text
GET /v1/infra/servers/:serverId/logs?lines=60
```

For a Node app, expect something like:

```text
npm run start
node dist/...
... listening on http://0.0.0.0:3000
```

3. External smoke test:

```text
POST /v1/infra/servers/:serverId/access-tokens { "mode": "api-bearer" }
GET <appUrl>/api/health
Authorization: Bearer <returned token>
```

Do not save the preview token into the repository. If it was written to a temporary file, delete it after the check.

## Auto-sleep

If the server "does not start", check the status first.

A common case:

```json
{
  "status": "sleeping",
  "blackholeStatus": "DISCONNECTED"
}
```

This is not a code error. The server fell asleep because of `sleepAfterMinutes`.

Options:

- wake it with another deploy;
- raise auto-sleep, for example to `240`;
- disable sleep if constant balance spending is acceptable.

## Typical Agent Mistakes

1. Mistake: thinking `vibe_app_...` is needed for deploy.
   - correct: infra-deploy uses `vibe_api_...`;
   - `vibe_app_...` is for the runtime app.

2. Mistake: asking the user for a new key when `/v1/me` says the personal key cannot bind placements.
   - correct: that is expected for `vibe_api_...`;
   - first create the OAuth app via `POST /v1/apps`;
   - take `rawKey` from the response;
   - save it as `VIBE_APP_KEY`;
   - only then work with placements.

3. Mistake: thinking a new personal API key is needed.
   - correct: first check `.env`, existing app/server ids, `/v1/me`, `/v1/apps`, logs, and server status.

4. Mistake: trying to publish to the catalog when the task is just to make the app visible.
   - correct: placements bind is enough for an MVP.

5. Mistake: deploying `.env` or `node_modules`.
   - correct: exclude `.env`, install dependencies on the server via `npm ci --omit=dev`.

6. Mistake: treating a sleeping server as a crashed app.
   - correct: check `status`, `blackholeStatus`, logs, and auto-sleep.

7. Mistake: leaving `work/*.json` with API responses in the deploy archive.
   - correct: exclude `work/`, and especially never store preview access tokens.

## Short Message for the Agent

No new personal key is needed. Use `VIBECODE_API_KEY` (`vibe_api_...`) from `.env` for the VibeCode infra API: create app/server, deploy/status/logs. If `/v1/me` says the personal key cannot bind placements / do transparent iframe auth, that is expected: this needs `vibe_app_...`, the raw key of the OAuth app. Do not ask the user for it right away — first create the OAuth app via `POST /v1/apps`, take `rawKey` from the response, and save it locally as `VIBE_APP_KEY`. `vibe_app_...` goes into the server env as `VIBE_APP_KEY` and is used for placements bind; it does not deploy infra. Build the deploy archive without `.env`, `.git`, `node_modules`, `coverage`, `work`; runtime `node20`, install `npm ci --omit=dev`, start `npm run start`, port `3000`. For visibility in Bitrix24, bind the `LEFT_MENU` and `CRM_COMPANY_DETAIL_TAB` placements through the VibeCode API with handler `https://vibecode.bitrix24.tech/v1/bitrix-handler`. If the server does not open, check `status` first: `sleeping` means auto-sleep, not a crashed app.

---

## Addendum: Verified in Practice

This section was added to the original note. Everything below was confirmed by real API calls, not read from documentation.

### bind Works Without User Consent, publish Does Not

This is the main point. Two different paths, and picking the wrong one leads to a dead end:

```text
POST /v1/placements/bind      X-Api-Key: vibe_app_...     -> works immediately
POST /v1/apps/:id/publish     -> 400 NO_USER_TOKEN
```

`bind` takes the dev-key path and needs only the app key. `publish` binds placements with a user OAuth token, which a fresh app does not have — and the only way to get one is a consent page in a browser. Publishing is only for the catalog; placement visibility does not need it.

**The documentation is wrong here.** In `/v1/me`, the `api._rules` field claims: "With an OAuth app key: POST /v1/placements/bind (needs placement scope + Bearer session)". No Bearer session is needed. When docs and practice disagree, try the endpoint.

### The Handler Creates the App Token

For a fresh app, `/v1/me` with its key shows `currentUser: null`. That is not a problem: the handler issues the OAuth token the first time an employee opens the placement. The scheme bootstraps itself — bind first, the rest follows.

This also means a new `vibe_app_` key from the dashboard will not help; it starts in exactly the same state. The key is not the blocker.

### publish Also Requires a Snapshot

If publish is genuinely needed, a fresh source snapshot must exist first, otherwise `409 SNAPSHOT_REQUIRED`:

```text
POST /v1/apps/:id/sources     Content-Type: application/gzip
```

The server snapshot that deploy saves automatically does not count — the app needs its own.

### Small Things That Cost Time

- `tar` on macOS puts `._*` and `.DS_Store` into the archive. Build with `COPYFILE_DISABLE=1`.
- `curl` treats `[` and `]` in a URL as a glob, so filters like `filter[companyId]=4` are sent empty. Use `-g` / `--globoff`.
- `accessPolicy` defaults to `OWNER_ONLY`, and the Gateway serves its own stub instead of the app. For an app behind the platform handler, the right value is `PORTAL`.
- Server creation accepts an `Idempotency-Key` — a retry after a lost response will not provision a second billable machine.
