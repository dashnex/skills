# Webapp Cron (Scheduled Jobs)

Scheduling cron jobs for the **application itself** — declared in the app's root
`dashnex.json`. For schedules that ship *inside a module*, see
[bindings.md](bindings.md) → Schedules.

## Webapp vs module schedules

- **Module schedules** use the plain-string form (`"schedules": ["*/5 * * * *"]`)
  in the module's own `dashnex.json`, and the module exports `scheduled` from its
  `./worker`. The build dispatches that cron to the module.
- **Webapp schedules** are different: the app worker `src/dashnex/worker.ts` is
  **auto-generated — never edit it by hand**. So the app registers cron with the
  **object form** `{ cron, handler }`, pointing at a handler file you own.

## 1. Declare the schedule in root `dashnex.json`

```json
{
  "schedules": [
    { "cron": "0 * * * *", "handler": "../modules/billing/cron/hourly.js" }
  ]
}
```

- `cron` — standard 5-field expression, evaluated in **UTC**.
- `handler` — used verbatim as a dynamic `import()` specifier inside the generated
  `src/dashnex/worker.ts`, so the path is **relative to `src/dashnex/`**. A
  `../modules/...` path therefore resolves to `src/modules/...`. Point it at the
  compiled `.js` (the `.ts` source lives alongside it).

## 2. Write the handler

```ts
// src/modules/billing/cron/hourly.ts
export async function scheduled(event: ScheduledController, env: Env, ctx: ExecutionContext) {
  // runs hourly; event.cron === '0 * * * *'
}
```

- Must export a `scheduled` function. The args `(event, env, ctx)` are passed but
  optional — `export async function scheduled() { ... }` is also valid.
- Worker context is initialized before dispatch, so `getBinding(...)`, the DB, and
  services all work normally inside the handler.

## 3. Build

```bash
npx dashnex build
```

This regenerates `src/dashnex/worker.ts`, wiring a dispatch block per schedule:

```ts
if (event.cron === '0 * * * *') {
  const { scheduled: localScheduled } = await import('../modules/billing/cron/hourly.js');
  if (localScheduled) crons.push(localScheduled(event, env, ctx));
}
```

Never hand-edit the generated worker — rerun `build` instead.

## Multiple jobs

Add more entries. Several handlers may share the same cron — each one fires:

```json
{
  "schedules": [
    { "cron": "*/5 * * * *", "handler": "../modules/payments/cron/billing.js" },
    { "cron": "*/5 * * * *", "handler": "../modules/payments/cron/auto-charge.js" }
  ]
}
```

## Cron syntax (UTC)

| Expression    | Runs                  |
|---------------|-----------------------|
| `* * * * *`   | every minute          |
| `*/5 * * * *` | every 5 minutes       |
| `0 * * * *`   | top of every hour     |
| `0 0 * * *`   | daily at 00:00 UTC    |
| `0 3 * * 1`   | Mondays at 03:00 UTC  |

## Local testing

```bash
npx dashnex dev
# in another shell, fire a scheduled run:
curl "http://localhost:8787/cdn-cgi/handler/scheduled?cron=0+*+*+*+*"
```

## Deploy & known limitation

On deploy, cron triggers are registered from `dashnex bindings --json` (schedules
merged across the app + all modules) and synced to the platform schedules service.

**The deploy-time merge currently forwards only plain-string cron entries.** The
object form `{ cron, handler }` is wired into the worker for local `build`/`dev`,
but is not yet propagated to production registration. So an app-level cron declared
this way runs under `dashnex dev` but may not fire in production until its trigger
is registered. After deploying, verify the schedule is active; if it isn't, register
the bare cron string through the deploy merge (or update `getMergedBindings` in
`@dashnex/core` to emit `cron` from object entries). See [bindings.md](bindings.md).
