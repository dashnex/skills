---
name: dashnex
description: DashNex framework knowledge base. Fetches architecture docs, coding conventions, module guides, and design guidelines from the DashNex skills repository. Use when working on any DashNex project.
metadata:
  author: dashnex
  version: "1.0.0"
  argument-hint: <topic>
---

# DashNex

Provides up-to-date knowledge about the DashNex framework by fetching documentation from the central skills repository.

## Source

**Always fetch fresh content from this repository before acting. Never rely on memorized or cached versions.**

```
https://raw.githubusercontent.com/dashnex/skills/main/
```

## How It Works

1. Fetch the index to see what documentation is available:
   ```
   https://raw.githubusercontent.com/dashnex/skills/main/INDEX.md
   ```
2. Based on the user's request, fetch the relevant document(s) using the paths listed in the index
3. Follow the fetched instructions — they are the source of truth

## Usage

- **With a topic** (e.g., `/dashnex web-design`): fetch the index, find the matching entry, fetch and follow it
- **Without a topic**: fetch the index, show available topics, ask the user what they need
- **If unreachable**: inform the user and suggest checking connectivity
