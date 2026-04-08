# Routes

Routes expose API endpoints from a module.

```ts
export interface Route {
  path: string;
  methods: string[];
  roles?: string[];
  guards?: DashnexGuard[];
  handler: (request: Request, params?: any) => Promise<Response | unknown> | Response | unknown;
}
```

## Parameters

- **path** — should start with `/api` followed by module name (e.g., `/api/contacts`).
- **methods** — HTTP methods to handle (e.g., `['GET', 'POST']`).
- **roles** — roles allowed to call the route. Setting this auto-applies `AuthGuard` and `CheckRolesGuard`. Use `["admin"]` for admin-only, `["user"]` for any authenticated user, `[]` or omit for public access.
- **guards** — custom access control functions.
- **handler** — receives `Request` and optional route params, returns a `Response` or plain object.

Place specific routes before parameterized ones (e.g., `/api/contacts/import` before `/api/contacts/:id`).

## Example

```ts
import { Route } from '@dashnex/types';

export const routes: Route[] = [
  {
    path: '/api/auth/logout',
    methods: ['POST'],
    roles: ['user'],
    handler: logout,
  },
  {
    path: '/api/user/:id',
    methods: ['GET'],
    roles: ['user'],
    guards: [
      async (request: Request, route: Route, routeParams: Record<string, any>) => {
        const user = await getUser(request);
        return user && (user.id == routeParams.id || hasRole(user, 'admin'));
      }
    ],
    handler: getUserById
  },
];
```
