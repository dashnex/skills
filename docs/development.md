# Development, Testing & Publishing

## Development Setup

### 1. Clone and install the Web Application

```bash
git clone <webapp-repo-url>
pnpm install
```

### 2. Link your module to the Web App

In your module folder:
```bash
pnpm link
```

In the Web App folder:
```bash
pnpm link @dashnex/<modulename>
```

This creates a symlink at `node_modules/@dashnex/<modulename>`.

### 3. Run development

Module folder:
```bash
pnpm dev
```

Web App folder:
```bash
npx dashnex build
pnpm dev
```

## UI Components

Import from `@dashnex/ui/client`:

```ts
import { Button, Input, Dialog } from '@dashnex/ui/client';
```

The Web Application has built-in Tailwind support. `@dashnex/ui` provides additional components.

## Testing

- Use `vitest` (preinstalled with module template).
- Place tests in the `tests/` folder.

```bash
pnpm test
```

## Publishing

### Prerequisites

1. All tests pass (`pnpm test`)
2. Correct `name`, `description`, and `repository.url` in `package.json`
3. `files` in `package.json` contains only necessary files
4. `dashnex.json` is correct (envs, bindings, roles)
5. Exports in `src/server.ts` and `src/client.ts` are correct

### Steps

1. Update `version` in `package.json` (semver)
2. Build: `pnpm build`
3. Publish: `npm publish`
