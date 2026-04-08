# DashNex Style Guide

## REST API Design

- Use proper HTTP methods: `GET` for reading, `POST` for creating, `PATCH` for partial updates, `PUT` for full updates, `DELETE` for deleting.
- Use standard HTTP methods for CRUD — do not add verb postfixes (e.g., use `DELETE /api/contacts/bulk` not `POST /api/contacts/bulk/delete`).
- Verb postfixes are acceptable for non-CRUD operations (e.g., `POST /api/contacts/:id/duplicate`).
- Follow RESTful URL structure:
  ```
  GET    /api/{module}/{entity}?type={...}              — list with filter
  GET    /api/{module}/{entity}/:id                     — get by ID
  GET    /api/{module}/{entity}/:id/{relation}          — list related
  POST   /api/{module}/{entity}                         — create
  PUT    /api/{module}/{entity}/:id                     — update
  DELETE /api/{module}/{entity}/:id                     — delete
  ```

## Response Format

Return plain objects from handlers — the framework wraps them automatically:

```ts
// returning data — framework wraps in { success: true, data: ... }
return entity
return { contacts, total, page }

// returning data with custom status
return { data: entity, status: 201 }

// returning success without data
return { success: true }
return { status: 204 }

// returning errors
return { error: 'Not Found', status: 404 }
return { error: 'Validation failed', status: 400 }
```

## Error Handling

- Do not use `try/catch` in route handlers — the framework's `core.error` listener handles errors centrally.
- **ZodErrors** are caught automatically and return `{ success: false, error: "<first issue>" }` with status `400`.
- For known error conditions, return an error object instead of throwing:
  ```ts
  if (!broadcast) {
    return { error: 'Broadcast not found', status: 404 };
  }
  ```

## Architecture / Layer Responsibilities

### Route Handlers (routes/)

- Extract request body, query params, route params.
- Call service methods with entity objects.
- Return plain objects.
- Must **not** contain business logic, Zod validation, or DB operations.

### Services (services/)

- Validate input using Zod schemas.
- Contain all database queries and business logic.
- Are singletons — import and call directly (e.g., `getContacts()`).

### MCP Tools & External API endpoints

- Thin wrappers that parse arguments and call service methods.
- Must **not** contain database operations or business logic.

## Function Parameters

- Pass entity objects to service methods, not IDs:
  ```ts
  // correct
  getContactFieldValues(contact)

  // incorrect
  getContactFieldValues(contact.id)
  ```
- Use proper types — prefer `Date` over `string` for dates.
- Avoid `any`, `unknown`, or `Record<string, unknown>`. Define explicit types.

## Naming

- Avoid type names in variables (use `emails` not `emailArray`).
- Service method names should be self-descriptive:
  ```ts
  // correct
  contactService.findTagsByIds(ids)
  service.deleteKey(key)

  // incorrect
  contactService.getDataByMultipleTagIdentifiers(ids)
  ```
- Methods returning `boolean` should read like questions: `hasAccess()`, `isActive()`.

## Query Design

- Prefer JOINs over post-processing.
- Use `COUNT(*)` — do not fetch all rows for `.length`.
- Avoid N+1 queries — use aggregation with JOINs and GROUP BY.
- Combine multiple similar queries into one using `SUM(CASE WHEN ... THEN 1 ELSE 0 END)`.
- Never use `sql.raw()` with string interpolation — always use the `sql` tagged template.
- Do not use transactions (D1 does not support them).
- Merge multiple migrations into a single file per feature.
- Do not use `CHECK` constraints for value lists — use `text` type instead.

## Return Types

- Return full entity objects. Do not cherry-pick columns:
  ```ts
  // correct
  return db.select().from(purchases)
    .innerJoin(offers, eq(offers.id, purchases.offerId))

  // incorrect
  return db.select({ purchaseId: purchases.id, offerName: offers.name })
    .from(purchases).innerJoin(...)
  ```
- For aggregated results, structure with entity objects:
  ```ts
  return { offer: { id, name }, activeCount: 5, cancelledCount: 2 }
  ```

## Authorization

- Do not add permission checks when the route-level role restriction already guarantees access.
- Role hierarchy: `owner` > `admin` > `user` — avoid redundant checks.

## Frontend Data Fetching

Do not use `fetch()` chains in components. Create client services:

```ts
// services/client/contacts.ts
export async function getContacts() {
  const res = await authFetch('/api/contacts');
  return res.json();
}

// In component
useEffect(() => {
  const load = async () => {
    const contacts = await getContacts();
    setContacts(contacts);
  };
  load();
}, []);
```

## Logging

Wrap all log statements in a debug check:

```ts
process.env.DEBUG && console.log(...)
```

## Code Organization

- Do not keep dead code or unused files.
- Export all models from the main entry point.
- Extract standalone functions out of classes when they don't belong.
- Add empty lines between logical blocks and before `return`.
