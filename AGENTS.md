# AGENTS.md

Guidance for Codex agents working in this repository.

## Repository Overview

This repository is a collection of assistant skills: structured workflow guides that teach AI assistants how to use specific MCP tool sets or local helper workflows.

Each skill is a self-contained directory with a `SKILL.md` entry point and optional supporting materials.

## Skill Structure

Use this directory convention for every skill:

```text
<skill-name>/
├── SKILL.md              # Main skill definition: YAML frontmatter + workflow guide
├── references/           # Detailed reference docs
├── assets/               # Supporting assets
└── scripts/              # Helper scripts
```

`SKILL.md` must start with YAML frontmatter:

```yaml
---
name: <skill-name>
description: <when to trigger this skill; used for skill matching>
---
```

Keep `SKILL.md` focused on trigger conditions, decision trees, workflow steps, tool usage patterns, and guardrails. Put long parameter references, examples, and implementation details under `references/`.

## Current Skills

- `kingbase-skill`: Guides interaction with KingBase databases, which are PostgreSQL-compatible. Covers querying, schema exploration, DML/DDL with confirmation, and performance analysis through `kb_*` MCP tools.
- `swagger-api-skill`: Guides interaction with Swagger/OpenAPI specs through `swagger_*` MCP tools. Covers spec loading, endpoint discovery, schema inspection, and API calls with preview/confirm behavior.
- `image2ppt`: Converts images, screenshots, diagrams, flowcharts, architecture charts, and infographics into PowerPoint/PPTX decks, including editable reconstructions with `python-pptx`.

## Repository Patterns

- Preserve two-phase confirmation for write operations. DML, DDL, and API calls should preview first, then execute only after explicit confirmation.
- Optimize for token efficiency. Skills should return or point to compact summaries and cache/reference files when full details are large.
- Define clear access control levels for potentially sensitive operations, such as `readonly`, `readwrite`, `full`, and `admin`.
- Prefer reusable helper scripts in a skill's `scripts/` directory over retyping large procedural code in the skill body.
- Keep reference docs local to the skill they support.

## Editing Guidelines

- When creating a new skill, create a dedicated directory with `SKILL.md` and add `references/`, `assets/`, or `scripts/` only when needed.
- Write frontmatter descriptions as practical trigger rules, not marketing copy. They should make it obvious when an assistant should load the skill.
- Start substantial skills with a workflow decision tree or equivalent routing section.
- Document tool parameters, common workflows, safety constraints, and failure modes.
- Keep changes scoped to the relevant skill unless updating shared repository documentation is part of the task.
- Preserve existing user-authored content and avoid unrelated formatting churn.

## Verification

For documentation-only changes, review the edited Markdown for structure, broken local links, and consistency with the affected skill directories.

For script changes, run the smallest relevant command that validates the script or example workflow before reporting completion.
