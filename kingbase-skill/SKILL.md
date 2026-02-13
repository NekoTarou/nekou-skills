---
name: kingbase-skill
description: Connect to and manage KingBase databases (PostgreSQL-compatible) using MCP tools. Use this skill when working with KingBase database operations including querying data, managing schemas, analyzing performance, or exploring database structures. Supports 11 database tools, 2 prompts for query assistance, and 2 resources for monitoring.
---

# KingBase MCP Skill

## Overview

This skill enables AI assistants to interact with KingBase databases (PostgreSQL-compatible enterprise database widely used in Chinese government and enterprise environments) through the Model Context Protocol (MCP).

KingBase MCP Server provides:
- **11 Database Tools**: Query, DML, DDL, schema inspection, statistics
- **2 Prompts**: Query helper and schema overview
- **2 Resources**: Database config and server status
- **Fine-grained Access Control**: `readonly` / `readwrite` / `full` / `admin`
- **Secure Confirmation**: Built-in safety mechanisms for write operations

## Quick Start

### Configuration

Add to your MCP configuration (`~/.claude.json` or `.mcp.json`):

```json
{
  "mcpServers": {
    "kingbase": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "your-db-host",
        "DB_PORT": "54321",
        "DB_USER": "system",
        "DB_PASSWORD": "your-password",
        "DB_NAME": "kingbase",
        "DB_SCHEMA": "public",
        "ACCESS_MODE": "readonly"
      }
    }
  }
}
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_HOST` | Database host | `localhost` |
| `DB_PORT` | Database port | `54321` |
| `DB_USER` | Username | `system` |
| `DB_PASSWORD` | Password | (empty) |
| `DB_NAME` | Database name | `kingbase` |
| `DB_SCHEMA` | Default schema | `public` |
| `ACCESS_MODE` | Permission level (see below) | `readonly` |
| `TRANSPORT` | Transport mode: `stdio` or `http` | `stdio` |

### Access Modes

| Level | Value | Allowed Operations |
|-------|-------|-------------------|
| Read-only | `readonly` (default) | SELECT queries, schema inspection |
| Read-write | `readwrite` | readonly + INSERT / UPDATE |
| Full | `full` | readwrite + DELETE |
| Admin | `admin` | full + DDL (CREATE / ALTER / DROP / TRUNCATE) |

**Always start with `readonly` and escalate as needed.**

## Core Tools Reference

### 1. Query Operations (Read-only)

**kb_query**: Execute read-only SQL queries (SELECT/WITH/SHOW)

```
Use kingbase_kb_query tool with:
- sql: "SELECT * FROM users WHERE status = $1", params: ["active"]
- sql: "SELECT count(*) FROM orders WHERE created_at > $1", params: ["2024-01-01"]
```

**kb_explain**: Analyze query execution plans for performance optimization

```
Use kingbase_kb_explain tool with:
- sql: "SELECT * FROM large_table WHERE status = 'pending'"
- analyze: true (for actual execution timing)
```

### 2. Schema Exploration (Read-only)

**kb_list_schemas**: List all database schemas
```
Use kingbase_kb_list_schemas tool (no parameters)
```

**kb_list_tables**: List tables and views in a schema
```
Use kingbase_kb_list_tables tool with:
- schema: "public" (optional, defaults to DB_SCHEMA)
- type: "table" or "view" or "all"
```

**kb_describe_table**: Get detailed table structure
```
Use kingbase_kb_describe_table tool with:
- table: "users"
- schema: "public" (optional)
```

**kb_list_indexes**: List indexes on a table
```
Use kingbase_kb_list_indexes tool with:
- table: "users"
- schema: "public" (optional)
```

**kb_list_constraints**: List constraints on a table
```
Use kingbase_kb_list_constraints tool with:
- table: "users"
- schema: "public" (optional)
```

### 3. Data Preview (Read-only)

**kb_table_data**: Preview table data with filtering and pagination
```
Use kingbase_kb_table_data tool with:
- table: "users"
- limit: 10 (default: 100, max: 1000)
- where: "status = 'active'" (optional)
- order_by: "created_at DESC" (optional)
- offset: 0 (optional)
```

**kb_table_stats**: Get table statistics (size, row count, etc.)
```
Use kingbase_kb_table_stats tool with:
- table: "users"
- schema: "public" (optional)
```

### 4. Data Modification (Write Operations)

**⚠️ Requires ACCESS_MODE >= readwrite**

**kb_execute**: Execute DML statements (INSERT/UPDATE/DELETE)

This tool uses **two-phase confirmation** for safety:
1. First call (without `confirmed: true`) → Returns preview, does NOT execute
2. Confirm call (with `confirmed: true`) → Actually executes

```
# Phase 1: Preview (do NOT add confirmed parameter)
Use kingbase_kb_execute with:
- sql: "UPDATE users SET status = $1 WHERE id = $2"
- params: ["inactive", 123]

# Phase 2: Execute (after user confirmation)
Use kingbase_kb_execute with:
- sql: "UPDATE users SET status = $1 WHERE id = $2"
- params: ["inactive", 123]
- confirmed: true
```

### 5. Schema Modification (DDL Operations)

**⚠️ Requires ACCESS_MODE = admin**

**kb_execute_ddl**: Execute DDL statements (CREATE/ALTER/DROP/TRUNCATE)

Also uses two-phase confirmation:

```
# Phase 1: Preview
Use kingbase_kb_execute_ddl with:
- sql: "CREATE TABLE logs (id SERIAL PRIMARY KEY, message TEXT)"

# Phase 2: Execute
Use kingbase_kb_execute_ddl with:
- sql: "CREATE TABLE logs (id SERIAL PRIMARY KEY, message TEXT)"
- confirmed: true
```

## Prompts

### kb_query_prompt
Helps construct SQL queries from natural language descriptions. Use this when you need to:
- Translate user requirements into SQL
- Generate complex queries with joins, aggregations, etc.
- Optimize existing queries

### kb_schema_overview
Provides comprehensive database structure analysis. Use this to:
- Get an overview of the entire database schema
- Understand relationships between tables
- Identify key tables and their purposes

## Resources

### kingbase://config
Returns current database connection configuration (without sensitive info like password). Use to verify connection settings.

### kingbase://status
Returns server runtime status including:
- Server version
- Connection status
- Uptime
- Active sessions

## Common Workflows

### Workflow 1: Explore Database Structure

1. `kb_list_schemas` → See all schemas
2. `kb_list_tables` with schema → See tables in a schema
3. `kb_describe_table` for interesting tables → Understand structure
4. `kb_table_stats` → Check table sizes

### Workflow 2: Analyze Query Performance

1. Write your query
2. `kb_explain` with `analyze: true` → See execution plan
3. Identify bottlenecks (full table scans, missing indexes)
4. `kb_list_indexes` on affected tables → Check existing indexes
5. Optimize query or suggest new indexes

### Workflow 3: Safe Data Modification

1. `kb_table_data` with WHERE clause → Verify which rows will be affected
2. `kb_execute` (Phase 1, no `confirmed`) → Preview the operation
3. **Get user confirmation**
4. `kb_execute` (Phase 2, `confirmed: true`) → Execute
5. `kb_table_data` again → Verify results

### Workflow 4: Generate Database Documentation

1. `kb_list_schemas` → Get all schemas
2. For each schema: `kb_list_tables` → Get tables
3. For each table: `kb_describe_table` → Get columns and constraints
4. Compile into documentation format

## Best Practices

### Security
- **Always start with `ACCESS_MODE: readonly`**
- Only escalate permissions when necessary
- Use parameterized queries (the `params` field) to prevent SQL injection
- For write operations, always use two-phase confirmation

### Performance
- Use `kb_explain` before running queries on large tables
- Add appropriate LIMIT clauses when exploring data
- Use `where` parameter in `kb_table_data` to filter large tables

### Query Patterns
- **Parameterized queries**: Use `$1, $2, ...` placeholders with `params` array
- **Schema qualification**: Table names without schema prefix are auto-qualified with `DB_SCHEMA`
- **Date filtering**: Use ISO 8601 format for dates in parameters

## Safety Features

### Two-Phase Confirmation
Both `kb_execute` (DML) and `kb_execute_ddl` (DDL) require explicit confirmation:

1. **First call** returns a preview showing:
   - SQL to be executed
   - Estimated affected rows
   - Confirmation prompt

2. **Confirmation call** with `confirmed: true` actually executes

This prevents accidental data modification.

### Access Mode Enforcement
- Tools check `ACCESS_MODE` before executing
- Write operations are blocked in `readonly` mode
- DDL operations require `admin` mode
- Clear error messages indicate required access level

## Troubleshooting

### Connection Issues
- Verify `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD` are correct
- Check firewall rules allow connection to database
- Use `kingbase://status` resource to check server status

### Permission Errors
- Error says "requires ACCESS_MODE" → Update `ACCESS_MODE` environment variable
- Remember: Changes to env vars require MCP server restart

### Query Errors
- Check SQL syntax (KingBase is PostgreSQL-compatible)
- Verify table/schema names exist (`kb_list_tables`)
- Use `kb_explain` to validate query structure

## References

For detailed query patterns and examples, see:
- `references/query-patterns.md` - Common SQL patterns and examples
- `references/configuration.md` - Advanced configuration options

## External Resources

- KingBase MCP Server: https://github.com/NekoTarou/kingbase-mcp-server
- KingBase Official: https://www.kingbase.com.cn/
- MCP Protocol: https://modelcontextprotocol.io/
