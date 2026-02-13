# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of **Claude Code skills** — structured workflow guides that teach AI assistants how to use specific MCP (Model Context Protocol) tool sets. Each skill is a self-contained directory with a `SKILL.md` entry point and supporting reference materials.

## Skill Structure

Each skill follows this directory convention:

```
<skill-name>/
├── SKILL.md              # Main skill definition (frontmatter + workflow guide)
├── references/           # Detailed reference docs (query patterns, config, tool params)
├── assets/               # Supporting assets
└── scripts/              # Helper scripts
```

### SKILL.md Format

Every `SKILL.md` starts with YAML frontmatter:
```yaml
---
name: <skill-name>
description: <when to trigger this skill — used for skill matching>
---
```

The body contains workflow decision trees, tool usage patterns, and best practices. Skills are designed to be loaded into an AI assistant's context to guide MCP tool usage.

## Current Skills

- **kingbase-skill** — Guides interaction with KingBase databases (PostgreSQL-compatible) via `kb_*` MCP tools. Covers query, schema exploration, DML/DDL with two-phase confirmation, and performance analysis.
- **swagger-api-skill** — Guides interaction with Swagger/OpenAPI specs via `swagger_*` MCP tools. Covers spec loading, endpoint discovery, schema inspection, and API calls with preview/confirm pattern. Uses a file-cache pattern where tools return summaries + file paths (must `Read` cache files for full details).

## Key Patterns Across Skills

- **Two-phase confirmation**: Write operations (DML, DDL, API calls) use a preview-then-confirm pattern. First call without `confirmed: true` returns a preview; second call with `confirmed: true` executes.
- **Token efficiency**: Skills are designed to minimize token usage. The swagger skill returns ~200 char summaries + cache file paths instead of inline content.
- **Access control levels**: Skills define graduated permission levels (e.g., readonly → readwrite → full → admin for KingBase).

## Writing New Skills

When creating a new skill:
1. Follow the existing directory structure (`SKILL.md` + `references/`)
2. Include YAML frontmatter with `name` and `description` (description is critical for skill matching)
3. Start with a workflow decision tree showing when to use which tools
4. Document tool parameters and common workflows
5. Extract detailed reference material into `references/` to keep `SKILL.md` focused on workflows
