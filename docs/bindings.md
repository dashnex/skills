# Bindings

Bindings connect modules to platform resources. Configured in `dashnex.json`.

## Resource Bindings

### DB

```json
{ "bindings": { "MODULE_DB": { "type": "DB", "description": "Module database" } } }
```

Usually not needed — `@dashnex/core` provides `CORE_DB` used throughout the app.

### KV

```json
{ "bindings": { "MODULE_KV": { "type": "KV", "description": "Key-value storage" } } }
```

### STORAGE

```json
{ "bindings": { "MODULE_STORAGE": { "type": "STORAGE", "description": "File storage" } } }
```

### Using Bindings

```ts
import { getBinding } from '@dashnex/core';

const kv = getBinding('MODULE_KV');
await kv.put('key', 'value');
const value = await kv.get('key');
```

## AI

```json
{ "bindings": { "MODULE_AI": { "type": "AI", "description": "AI support" } } }
```

```ts
// Basic usage
const AI = await getBinding("AI");
const result = await AI.run("@cf/meta/llama-3.2-1b-instruct", { query: "Hello" });

// Recommended: using AI SDK
import { createWorkersAI } from "workers-ai-provider";
import { generateText } from "ai";

const binding = await getBinding("AI");
const ai = createWorkersAI({ binding });
const model = ai("@cf/meta/llama-2-7b-chat-int8");

const result = await generateText({
  model,
  prompt: "Write a poem about hello world.",
});
```

## RAG

```json
{ "bindings": { "MODULE_RAG": { "type": "RAG", "description": "RAG service" } } }
```

Methods:
- **Files**: `head(key)`, `get(key)`, `put(key, data, options)`, `delete(key)`, `list()`, `all()`
- **Search**: `search(params)`, `aiSearch(params)`

## Browser

```json
{ "bindings": { "MODULE_BROWSER": { "type": "BROWSER", "description": "Browser for web extraction" } } }
```

```ts
const response = await env.MODULE_BROWSER.fetch('https://example.com');
const image = await env.MODULE_BROWSER.screenshot('https://example.com');
const pdf = await env.MODULE_BROWSER.pdf('https://example.com');
const html = await env.MODULE_BROWSER.content('https://example.com');
```

## Queues

```json
{ "bindings": { "MODULE_QUEUE": { "type": "QUEUE", "description": "Background queue" } } }
```

Sending:
```ts
const queue = await getBinding('MODULE_QUEUE');
await queue.send({ url: request.url, method: request.method });
```

Processing in `src/worker.ts`:
```ts
export default {
  async queue(batch: MessageBatch, env: Env, ctx: ExecutionContext) {
    for (const msg of batch.messages) {
      console.log(`Message received: ${msg.body}`);
      msg.ack();
    }
  }
}
```

Make sure `./worker` is exported from `package.json` and run `npx dashnex build`.

## Schedules

In `dashnex.json`:
```json
{ "schedules": ["*/1 * * * *", "0 0 * * *"] }
```

Handler in `src/worker.ts`:
```ts
export default {}

export async function scheduled(event: ScheduledController, env: Env, ctx: ExecutionContext) {
  console.log(`Trigger fired for schedule: ${event.cron}`);
}
```

Make sure `./worker` is exported from `package.json` and run `npx dashnex build`.
