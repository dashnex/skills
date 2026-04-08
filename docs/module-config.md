# Module Configuration

## File Structure

Every `@dashnex` module follows this layout:

```
src/
  server.ts          # Server-side module config export
  client.ts          # Client-side module config export (pages, listeners)
  routes/
    index.ts         # Route definitions array
    <entity>.ts      # Handler functions per entity
  services/
    <entity>.service.ts
  models/
    <entity>.model.ts
  schemas/
    <entity>.schema.ts
  listeners/
    <event-name>.ts
  mcp-tools/         # Optional
    <tool-name>.ts
dashnex.json         # Module config (bindings, env)
package.json
```

## ModuleConfig Interface

```ts
export interface ModuleConfig {
  name: string;
  version: string;
  description?: string;
  roles?: Record<string, string[]>;
  env?: string[];
  menu?: AuthMenuItem[];
  bindings?: Record<string, { type: string; description?: string }>;
  pages?: DashnexAuthPage[];
  routes?: DashnexAuthRoute[];
  listeners?: EventListenerConfig[];
}
```

## server.ts

Server-side entry point — includes routes, listeners, and re-exports services/models/schemas for cross-module use:

```ts
import { ModuleConfig } from '@dashnex/types';
import { routes } from './routes/index.js';
import packageJson from '../package.json' with { type: "json" };
import dashnexConfig from '../dashnex.json' with { type: "json" };

export { getContacts } from './services/contact.service.js';
export { contacts, type Contact, type NewContact } from './models/contact.model.js';

export default {
  name: packageJson.name,
  version: packageJson.version,
  description: packageJson.description,
  ...dashnexConfig,
  listeners: [
    { event: "core.before_email_render", handler: onBeforeEmailRender },
    { event: "contacts.contact_created", handler: onContactCreated, priority: 100 },
  ],
  routes
} as ModuleConfig;
```

## client.ts

Client-side entry point — includes pages, client-side listeners, and re-exports React components:

```ts
import packageJson from '../package.json' with { type: "json" };
import dashnexConfig from '../dashnex.json' with { type: "json" };
import { pages } from './pages/index.js';

export { MyComponent } from './components/MyComponent.js';

export default {
  name: packageJson.name,
  version: packageJson.version,
  description: packageJson.description,
  ...dashnexConfig,
  pages
};
```

## dashnex.json

Declares bindings and environment variables:

```json
{
  "bindings": {
    "CORE_DB": { "type": "DB", "description": "Main database" },
    "UPLOADS": { "type": "STORAGE", "description": "File storage" }
  },
  "env": ["API_KEY", "WEBHOOK_SECRET"],
  "variables": [
    {
      "name": "limit",
      "description": "Defines the limit",
      "defaultValue": 10,
      "pattern": "[0-9]+",
      "options": [0, 10, 20]
    }
  ]
}
```

Binding types: `DB`, `STORAGE`, `KV`, `SERVICE`, `OBJECT`, `AGENT`, `WORKER`, `AI`, `RAG`, `QUEUE`.

## package.json

Key fields:

```json
{
  "name": "@dashnex/template",
  "exports": {
    ".": {
      "types": "./dist/server.d.ts",
      "default": "./dist/server.js"
    },
    "./client": {
      "types": "./dist/client.d.ts",
      "default": "./dist/client.js"
    },
    "./commands": {
      "types": "./dist/commands/index.d.ts",
      "default": "./dist/commands/index.js"
    }
  },
  "files": ["dist", "migrations", "dashnex.json"],
  "peerDependencies": {
    "vinext": ">=1",
    "react": ">=18",
    "react-dom": ">=18"
  }
}
```
