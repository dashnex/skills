# Events

Modules can emit their own events and listen to events from other modules. Listeners receive a context object that can be mutated in place.

## Listener Configuration

### server.ts
```ts
export default {
  listeners: [
    {
      event: 'core.request',
      // priority: -10,
      handler: async (context: CoreContext, event: CoreEventName) => {
        console.log(`${event} received`);
      }
    }
  ]
}
```

### client.ts
```ts
export default {
  listeners: [
    {
      event: 'core.after_page_load',
      handler: async (context: AuthPageContext, event: CoreEventName) => {
        console.log(`${event} received with params ${JSON.stringify(context.params)}`);
      }
    }
  ]
}
```

## Multiple and Wildcard Events

```ts
// Multiple events
{ event: ['core.request', 'core.response'], handler: myHandler }

// Wildcard
{ event: 'core.*', handler: myHandler }

// Combined
{ event: ['core.*', 'emails.send'], handler: myHandler }
```

## Priority

Optional `priority` controls execution order. Default is `0`. Higher values run first. Negative values run after default.

## Core Event Types

```ts
export type CoreEventName = 'core.request' | 'core.route' | 'core.params' | 'core.response' | 'core.error';

export interface CoreContext extends Context {
  request: Request;
  route?: Route;
  params?: Record<string, string>;
  response?: Response;
  error?: Error;
}

export type AuthPageContext = {
  user: any;
  page?: Page;
  params?: any;
  searchParams: Promise<any>;
}
```

## Common Framework Events

- `core.request`, `core.route`, `core.params`, `core.response`, `core.error`
- `core.before_email_render`, `core.menu_get`
- `system.admin_settings_get`, `system.user_account_get`
- `system.widgets_get`, `system.layouts_get`
- `system.before_page_load`, `system.after_page_load`
- `auth.user_format`
- `mcp.tools_get`, `mcp.resources_get`, `mcp.prompts_get`
- `api.routes_get`

## Triggering Custom Events

```ts
import { getEvents } from '@dashnex/core';
import { Context } from '@dashnex/types';

interface CustomContext implements Context {
  value?: string;
  enough?: boolean;
}

const events = getEvents();

// Register
events.on('module.event', (context: CustomContext) => {
  context.enough = true;
  context.value = 'anything';
});

// Emit
const context = {} as CustomContext;
await events.emit('module.event', context);

// With stop condition
const shouldStop = (ctx: CustomContext) => ctx.enough;
await events.emit('module.event', context, shouldStop);
```
