# Database Schema, Models & Migrations

DashNex uses Drizzle ORM with SQLite (D1). Each module can define its own models and migrations.

## Models

Models are located in `src/models/`. Define tables with automatic type inference:

```ts
import { sql } from 'drizzle-orm';
import { sqliteTable, text, integer, index } from 'drizzle-orm/sqlite-core';
import { createId } from '@paralleldrive/cuid2';

export const migrations = sqliteTable('migrations', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  filename: text('filename').notNull().unique(),
  module: text('module'),
  appliedAt: integer('applied_at', { mode: 'timestamp' }).default(sql`CURRENT_TIMESTAMP`)
}, (table) => [
  index('idx_migrations_filename').on(table.filename),
  index('idx_migrations_module').on(table.module)
]);

export type Migration = typeof migrations.$inferSelect;
export type NewMigration = typeof migrations.$inferInsert;
```

### Conventions

- Use CUID2 for primary keys (`createId()`).
- Use `date()` from `@dashnex/core` for date fields.
- Column names use `snake_case` in the database.
- Define indexes for frequently queried columns.
- Use `.mode('boolean')` for integer-backed booleans.
- Export both `$inferSelect` (read type) and `$inferInsert` (write type).

## Migrations

Place SQL migration files in the `migrations/` folder:

```sql
-- migrations/001_initial_schema.sql
CREATE TABLE IF NOT EXISTS migrations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  filename TEXT NOT NULL UNIQUE,
  module TEXT,
  applied_at INTEGER DEFAULT (strftime('%s', 'now'))
);

CREATE INDEX IF NOT EXISTS idx_migrations_filename ON migrations(filename);
CREATE INDEX IF NOT EXISTS idx_migrations_module ON migrations(module);
```

Run migrations from the Web App folder:

```bash
npx dashnex db migrate
```

### Migration Rules

- Do not use `CHECK` constraints for value lists — use `text` type instead.
- Merge multiple migrations into a single file per feature/PR.
- Do not use transactions (D1 does not support them).
