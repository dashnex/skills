# Pages

Modules can register React components to be rendered at specific browser paths.

```ts
export interface DashnexPage {
  path: string;
  title: string;
  description?: string;
  roles?: string[];
  component: ({ params }: { params?: Promise<any> }) => React.ReactElement;
}
```

## Parameters

- **path** — where the component renders. Should not start with `/api`. Use module name in the path (e.g., `/auth/login`, `/debug/`).
- **title** — page title.
- **description** — optional page description.
- **roles** — limits access based on user role.
- **component** — React component to render.

## Example

```tsx
// src/pages/Config.tsx
'use client';
import { use } from '@dashnex/core/client';

export default function Config({ params }) {
  const { id } = use(params)
  return <div>Configuration ID: {id}</div>
}
```

```ts
// src/pages/index.ts
import Config from './Config'

export const pages: DashnexPage[] = [
  {
    path: '/template/config/:id',
    title: 'Configuration',
    description: 'Configuration page for Template Module',
    component: Config,
  }
]
```
