# DashNex Skills

Central knowledge base for the DashNex framework. Provides documentation and agent skills used when working on any DashNex project.

## Contents

- **[INDEX.md](INDEX.md)** — Index of all available documentation topics
- **[docs/](docs/)** — Framework documentation (architecture, conventions, module guides)
- **[skills/](skills/)** — Agent skills that consume this repository

## Using the documentation

Documents are fetched directly from GitHub so they are always up to date:

```
https://raw.githubusercontent.com/dashnex/skills/main/{path}
```

Start from [INDEX.md](INDEX.md) to discover what is available, then fetch the specific topic you need.

## Using the `dashnex` skill

The [`dashnex`](skills/dashnex/SKILL.md) skill exposes this knowledge base to any agent runtime that supports the skill format (Claude Code, Cursor, and other LLM tools that load `SKILL.md` files). Invoke it with a topic to pull the relevant guide:

```
/dashnex <topic>
```

Without a topic, the skill lists available entries from the index and asks which one to load.

## Contributing

1. Add or edit a document under [`docs/`](docs/).
2. Register it in [INDEX.md](INDEX.md) with a short description.
3. Commit and push to `main` — consumers fetch from `main` directly, so changes take effect immediately.
