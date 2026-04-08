# DashNex Overview

## Glossary

**Module** — an NPM package under the `@dashnex` scope. Modules are isolated, self-sufficient units that provide defined functionality (pages, API routes, events, database models, CLI commands, MCP tools). They can be developed and shared by different teams without conflicts.

**Web Application** — a vinext application configured to work with `@dashnex` modules. It includes a wildcard route, page handlers, and `@dashnex/core` which bootstraps the database connection and loads configuration from all installed modules.

## Available Modules

| Package | Description |
|---------|-------------|
| `@dashnex/types` | Type definitions and interfaces |
| `@dashnex/ui` | Reusable UI components and design system |
| `@dashnex/cli` | Command-line interface tools |
| `@dashnex/core` | Core functionality and utilities |
| `@dashnex/users` | User management |
| `@dashnex/auth` | Authentication and authorization |
| `@dashnex/events` | Event tracking system |
| `@dashnex/system` | Shared elements |
| `@dashnex/mcp` | MCP (Model Context Protocol) server and tools |
| `@dashnex/contacts` | Contact management |
| `@dashnex/emails` | Email campaigns, templates, and transactional emails |
| `@dashnex/products` | Product and variant management |
| `@dashnex/offers` | Offers, checkouts, purchases, and subscriptions |
| `@dashnex/portal` | Customer portal |
| `@dashnex/affiliates` | Affiliate management |
| `@dashnex/analytics` | Analytics and reporting |
| `@dashnex/api` | Public API module |
| `@dashnex/oauth2-server` | OAuth2 server implementation |
| `@dashnex/debug` | Debugging and logging utilities |
| `@dashnex/example` | Example module template |

## Commands

```bash
pnpm install          # Install deps + dashnex postinstall + dashnex build
pnpm dev              # Start dev server
npx dashnex build     # Regenerate module registry
npx dashnex db migrate # Run database migrations
```
