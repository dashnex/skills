# DashNex Best Practices

## Route Handlers

Handlers are thin — they deal only with the HTTP layer:

- Extract request body, query params, and route params.
- Identify the current user from the request context.
- Call the appropriate service method with entity objects.
- Return the response (plain object, handled by the framework's response listener).

Handlers must **not** contain business logic, Zod validation, or database operations.

```ts
// GET with query params
export async function listContacts(request: Request) {
  const url = new URL(request.url);
  const page = parseInt(url.searchParams.get('page') || '1');
  const search = url.searchParams.get('search') || '';

  const service = await getContacts();
  return service.filterContacts({ page, search });
}

// GET with path params
export async function getContactById(_request: Request, { id }: { id: string }) {
  const service = await getContacts();
  const contact = await service.getContactById(id);
  if (!contact) return { error: 'Not found', status: 404 };
  return contact;
}

// POST with body
export async function createContact(request: Request) {
  const body = await request.json();
  const service = await getContacts();
  return service.createContact(body);
}
```

## Services (Singleton Pattern)

Services own all business logic and database access. They are singletons stored in `globalThis`:

```ts
import { getDatabase } from '@dashnex/core';
import { eq } from 'drizzle-orm';
import { contacts, type Contact } from '../models/contact.model.js';
import { contactSchema } from '../schemas/contact.schema.js';
import { getEvents } from '@dashnex/core';

declare global {
  var DASHNEX_CONTACTS: ContactService | undefined;
}

export const getContacts = async (): Promise<ContactService> => {
  if (!globalThis.DASHNEX_CONTACTS) {
    globalThis.DASHNEX_CONTACTS = new ContactService();
  }
  return globalThis.DASHNEX_CONTACTS;
};

export class ContactService {
  async createContact(data: ContactInput): Promise<Contact> {
    const validated = contactSchema.parse(data);
    const db = await getDatabase();

    const [contact] = await db
      .insert(contacts)
      .values(validated)
      .returning();

    await getEvents().emit('contacts.contact_created', { contact });
    return contact;
  }

  async getContactById(id: string): Promise<Contact | null> {
    const db = await getDatabase();
    const [contact] = await db
      .select()
      .from(contacts)
      .where(eq(contacts.id, id));
    return contact || null;
  }
}
```

- `getDatabase()` is called per method, not in the constructor.
- Validate input with Zod at the service boundary.
- Emit events after mutations.
- Return typed entity objects.
- For bulk operations on D1, chunk by 50 due to SQL variable limits.

## Models (Drizzle ORM)

```ts
import { date } from '@dashnex/core';
import { sqliteTable, text, integer, index } from 'drizzle-orm/sqlite-core';
import { createId } from '@paralleldrive/cuid2';

export const contacts = sqliteTable('contacts', {
  id: text('id').primaryKey().$defaultFn(() => createId()),
  email: text('email').notNull().unique(),
  firstName: text('first_name'),
  type: text('type').notNull().default('lead'),
  emailsAllowed: integer('emails_allowed', { mode: 'boolean' }).notNull().default(true),
  createdAt: date('created_at').$defaultFn(() => new Date()),
  updatedAt: date('updated_at').$defaultFn(() => new Date()),
}, (table) => [
  index('idx_contacts_email').on(table.email),
]);

export type Contact = typeof contacts.$inferSelect;
export type NewContact = typeof contacts.$inferInsert;
```

- Use CUID2 for primary keys.
- Use `date()` from `@dashnex/core` for date fields.
- Column names use `snake_case` in the database.
- Export both `$inferSelect` and `$inferInsert` types.

## Schemas (Zod)

```ts
import { z } from 'zod';

export const contactSchema = z.object({
  firstName: z.string().optional(),
  email: z.string().email(),
  type: z.enum(['lead', 'customer', 'vip']).default('lead'),
});

export type ContactInput = z.infer<typeof contactSchema>;
```

Schemas are used in services, not in handlers.

## Event Listeners

Listeners receive and mutate a typed context object:

```ts
export async function onBeforeEmailRender(context: CoreEmailContext) {
  const service = await getContacts();
  context.to = await Promise.all(context.to.map(async (recipient) => {
    if (recipient.id && !recipient.email) {
      const contact = await service.getContactById(recipient.id);
      if (contact) {
        return { name: `${contact.firstName} ${contact.lastName}`.trim(), address: contact.email };
      }
    }
    return recipient;
  }));
}
```

- Listeners mutate the context object in place.
- Use `priority` to control execution order (higher = runs first).

## MCP Tools

```ts
context.tools.push({
  name: 'contacts_create',
  description: 'Create a new contact',
  inputSchema: {
    type: 'object',
    properties: { email: { type: 'string' } },
    required: ['email'],
  },
});

context.toolHandlers.set('contacts_create', async (args) => {
  const service = await getContacts();
  const contact = await service.createContact(args);
  return {
    content: [{ type: 'text', text: JSON.stringify({ success: true, contact }) }]
  };
});
```

- No DB operations in tool handlers — call services.
- Response format: `{ content: [{ type: 'text', text: JSON.stringify(result) }] }`.

## Cross-Module Imports

Modules re-export their services from `server.ts`. Other modules import by package name:

```ts
import { getContacts, type Contact } from '@dashnex/contacts';

const contactService = await getContacts();
const contact = await contactService.getContactById(id);
```

## Request Lifecycle

```
Request → core.request → core.route → core.params → handler() → core.response → Response
                                                                      ↓ (on error)
                                                                  core.error
```

## ESM Imports

Always use ESM syntax. All relative imports must include `.js` extension:

```ts
// correct
import { Component } from '../components/Component.js'

// incorrect
const a = require('file')
import { Component } from '../components/Component'  // missing .js
```

## Dependencies

Put `vinext` and `react` into `peerDependencies` to avoid duplicate instances:

```json
{
  "peerDependencies": {
    "vinext": ">=1",
    "react": ">=18",
    "react-dom": ">=18"
  }
}
```
