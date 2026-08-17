---
name: bitrix24-development-en
description: "Helps in English to build Bitrix24 applications: choosing between a local app and a platform app, embedding placements into CRM cards, reading portal data over REST, and deploying to VibeCode. Use for English requests about Bitrix24 apps, placements, the BX24 JS SDK, CRM data, and VibeCode deploy."
---

# Bitrix24 Development

Use this skill when the user builds an application for Bitrix24: a tab inside a CRM card, a left-menu item, a widget, or any code that reads portal data.

## Choose the Architecture First

This is the first decision, and it determines whether the official BX24 JS SDK can be used at all. Getting it wrong costs a rewrite of the data layer, not a two-line fix.

| | Local app | Platform app (VibeCode) |
|---|---|---|
| Handler | your URL | fixed by the platform |
| BX24 JS SDK | works | **does not work** |
| Own backend | not needed | required (BFF) |
| Installation | manual in the portal UI | fully scriptable |

Ask the user before starting. A common conflict: the spec says "official BX24 JS SDK" and "no backend", while deployment is planned on the platform. Those are incompatible, and it is much cheaper to learn that before writing code.

## Platform App

The handler is always the platform's and cannot be changed. It receives the placement from Bitrix24 and redirects the iframe to `appUrl`, carrying only `placement`, `member_id`, and `placement_options`. `DOMAIN`, `AUTH_ID`, and `APP_SID` are lost.

Consequences:

- `BX24.init()` cannot handshake and `BX24.placement.info()` returns `{placement:null, options:null}`. Do not use the SDK.
- Read the card id from the query: `JSON.parse(params.get('placement_options')).ID`.
- The backend reads data: the Gateway injects `X-Vibe-Authorization`, and the app calls the API with the app key. Never return that token to the browser.
- Portal visibility needs **bind**, not **publish**. Details in `references/vibecode-deploy.md`.

## Local App

Registered in the portal: **Applications → For developers → Other → Local application**, with "uses API only" unchecked. Bitrix24 passes `DOMAIN`/`AUTH_ID`/`APP_SID` itself, so the SDK works natively and no backend is needed.

- `BX24.callMethod` proxies REST through the parent frame, so tokens never leave the portal.
- The frontend binds placements on the install page: `placement.bind` per placement, then `BX24.installFinish()`. Call `unbind` before `bind` to make reinstallation idempotent.
- The handler is opened with **POST**, not GET. Static hosting that answers only GET returns 405 — the server must serve the page for both methods.
- Do not set `X-Frame-Options`: the portal's iframe loads the page.

## Portal Data

- Deal semantics live in `stageSemanticId`: `S` won, `F` lost, `P` in progress. Do not rely on stage names — they differ per portal.
- Do not hardcode magic numbers (task `status`, priorities); the entity's `fields` endpoint returns the decoding.
- Deal, contact, and company titles are untrusted CRM data. Escape them before inserting into markup.

### Currencies

The portal's currency list gives the rate as `AMOUNT` per `AMOUNT_CNT` units, expressed in the **portal's base currency**.

```text
rate(currency) = AMOUNT / AMOUNT_CNT
value_in_base  = value * rate(deal currency)
value_in_target = value_in_base / rate(target currency)
```

Accumulate in the base currency and convert only on output — otherwise deals in different currencies sum incorrectly. Do not assume the base currency is any particular one; the formula above does not depend on it. A deal in a currency missing from the list cannot be converted — do not hide that, show a counter of skipped deals.

## Verification

A dashboard inside the portal cannot be opened from outside: the session exists only when the placement is really opened. So:

1. Compute the expected numbers from portal data over REST in advance, and compare them with what the UI shows.
2. Test the logic in a browser harness with a mocked data source, using real portal data.
3. Exercise the states: loading, error, empty, missing rate, currency without a rate, opened outside a card, and a title containing HTML.
4. The final run in a live card belongs to the user — say honestly that you have not seen it yourself.

## Text Style

- In explanations and examples, do not overuse lists: use at most three items by default. Longer lists are acceptable for required rules, states, checks, or API parameters when shortening would break the meaning.

## Development Rules

- Before implementation, follow the 5 professional vibe-coding rules: strict typing, required linting, tests for every change, coverage measurement, and input validation with dependency control.
- For Bitrix24 apps, pay special attention to data boundaries: placement query parameters, REST responses, CRM titles, currencies, stages, and any values coming from the portal.
- If the stack or platform blocks one rule, state the compromise and add the closest guardrail: a schema, runtime validator, smoke test, linter config, or explicit check.

## Materials

- VibeCode deploy and keys note: `references/vibecode-deploy.md`
