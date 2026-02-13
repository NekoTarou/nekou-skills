# Swagger MCP Tools Reference

## Tool Quick Reference

| Tool | Purpose | Requires Cache |
|------|---------|---------------|
| `swagger_load_spec` | Load spec from URL → parse → cache | No |
| `swagger_update_cache` | Re-fetch and rebuild cache | No |
| `swagger_get_info` | API metadata (title, version, servers, auth) | Yes |
| `swagger_list_tags` | Tags with endpoint counts | Yes |
| `swagger_list_paths` | Endpoints with filtering (tag, method, keyword) + pagination | Yes |
| `swagger_get_endpoint` | Endpoint summary + cache file path | Yes |
| `swagger_list_schemas` | Schema definitions with filtering + pagination | Yes |
| `swagger_get_schema` | Schema summary + cache file path | Yes |
| `swagger_search` | Full-text search across endpoints and schemas | Yes |
| `swagger_call_api` | Execute HTTP requests (2-phase: preview → confirm) | Yes |
| `swagger_set_auth` | Set/clear Authorization header | No |

## Tool Parameters

### swagger_load_spec
- `url` (required): URL to Swagger/OpenAPI spec (JSON or YAML)
- `headers` (optional): Custom headers object for authenticated spec URLs

### swagger_list_paths
- `tag` (optional): Filter by tag name
- `method` (optional): Filter by HTTP method (GET, POST, etc.)
- `keyword` (optional): Filter by keyword in path or summary
- `page` (optional): Page number for pagination
- `pageSize` (optional): Items per page

### swagger_get_endpoint
- `method` (required): HTTP method
- `path` (required): Endpoint path (e.g., `/users/{id}`)

### swagger_list_schemas
- `keyword` (optional): Filter by name
- `page` / `pageSize` (optional): Pagination

### swagger_get_schema
- `name` (required): Schema name

### swagger_search
- `keyword` (required): Search term
- `scope` (optional): `endpoints`, `schemas`, or `all` (default)

### swagger_call_api
- `method` (required): HTTP method
- `path` (required): Endpoint path
- `parameters` (optional): Query/path parameters object
- `body` (optional): Request body object
- `headers` (optional): Additional headers
- `confirmed` (required): `false` for preview, `true` to execute

### swagger_set_auth
- `token` (optional): Authorization header value (used as-is, no auto "Bearer" prefix)
- `clear` (optional): Set `true` to remove current token

## Cache File Structure

```
.swagger-cache/
├── meta.json              # URL, counts, timestamp
├── info.json              # Title, servers, security schemes
├── tags.json              # Tag list with counts
├── paths-index.json       # Endpoint index (method, path, summary, tags, cacheFile)
├── schemas-index.json     # Schema index (name, type, propertyCount, cacheFile)
├── endpoints/             # One JSON per endpoint (fully resolved)
│   └── GET__users__{id}.json
└── schemas/               # One JSON per schema (fully resolved)
    └── User.json
```

Tools like `swagger_get_endpoint` and `swagger_get_schema` return ~200 chars (summary + file path) instead of full content. Use the `Read` tool on the returned file path to get complete details.
