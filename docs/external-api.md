# External API

The `@dashnex/api` module provides an external API layer for third-party integrations with API key authentication, role-based authorization, dynamic route registration, and automatic OpenAPI documentation.

## Overview

- All external API endpoints are mounted under `/api/v1/`
- Authentication via Bearer API key in the `Authorization` header
- Other modules register endpoints via the `api.routes_get` event
- OpenAPI 3.0 spec auto-generated at `GET /api/v1/docs.json`

## Request Flow

```
External Request → Matches /api/v1/* prefix?
  ├─ No → normal routing
  └─ Yes → Emit api.routes_get → Route matching
       → ApiKeyAuthGuard (401 if invalid)
       → ApiKeyRolesGuard (403 if insufficient)
       → Custom guards → Execute handler → JSON response
```

## Registering Endpoints

Listen to `api.routes_get`:

```ts
import type { ApiRoutesGetContext } from '@dashnex/api'
import { z } from 'zod'

export async function onApiRoutesGet(context: ApiRoutesGetContext) {
  context.routes.push({
    path: '/projects',
    methods: ['GET'],
    roles: ['projects:read'],
    handler: async (request) => {
      const service = await getProjects()
      return service.listAll()
    },
    summary: 'List all projects',
    tag: 'Projects',
    responses: { 200: z.array(projectSchema) },
  })

  context.routes.push({
    path: '/projects',
    methods: ['POST'],
    roles: ['projects:write'],
    handler: async (request) => {
      const body = await request.json()
      const service = await getProjects()
      return service.create(body)
    },
    summary: 'Create a project',
    tag: 'Projects',
    requestBody: createProjectSchema,
    responses: { 200: projectSchema },
  })
}
```

## Route Definition

```ts
interface ExternalApiRoute extends AuthRoute {
  summary?: string;
  description?: string;
  tag?: string;
  parameters?: OpenApiParameter[];
  requestBody?: ZodType;
  responses?: Record<number, ZodType>;
}
```

## Defining Roles

In `dashnex.json`:

```json
{
  "roles": {
    "projects:write": ["projects:read"],
    "projects:read": []
  }
}
```

## Types

```ts
import type { ApiRoutesGetContext, ExternalApiRoute } from '@dashnex/api'
```
