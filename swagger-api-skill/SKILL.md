---
name: swagger-api-skill
description: Workflow guide for using the Swagger API MCP Server to explore, understand, and call REST APIs from Swagger/OpenAPI specs. Use when the user wants to load an API spec, browse endpoints, understand data models, search for specific operations, or make API calls via the swagger_* MCP tools. Triggers on any task involving Swagger/OpenAPI specs, API exploration, API integration, or when swagger_* tools are available.
---

# Swagger API Skill

Use the `swagger_*` MCP tools to load, explore, and call REST APIs from Swagger 2.0 / OpenAPI 3.x specs. Tools return compact summaries + cache file paths — always use the `Read` tool on returned file paths to get full details.

For tool parameters and cache structure, see [references/tools-reference.md](references/tools-reference.md).

## Workflow Decision Tree

```
User request
├── "Load/explore this API" → Loading Workflow
├── "Find endpoint for X" → Search Workflow
├── "Call this API" → API Call Workflow
├── "What does this schema look like?" → Schema Workflow
└── "Help me integrate with this API" → Full Integration Workflow
```

## Loading Workflow

1. `swagger_load_spec({ url })` — load and cache the spec
2. `swagger_get_info` — show title, version, servers, auth schemes
3. `swagger_list_tags` — show resource groups with counts
4. `swagger_list_paths` — overview of all endpoints (paginate with `page`/`pageSize` if many)
5. `swagger_list_schemas` — overview of all data models
6. Summarize: purpose, resource groups, auth requirements

**Important**: If the spec URL requires authentication, pass custom `headers` to `swagger_load_spec`.

## Search Workflow

1. `swagger_search({ keyword })` — find matching endpoints and schemas
2. For relevant endpoints: `swagger_get_endpoint({ method, path })` → get summary + file path
3. `Read` the cache file path to see full parameters, request body, and responses
4. For referenced schemas: `swagger_get_schema({ name })` → `Read` the cache file
5. Summarize usage: required params, expected responses, related endpoints

**Tip**: Use `swagger_list_paths({ tag, method, keyword })` for filtered browsing when search is too broad.

## API Call Workflow

1. Ensure spec is loaded (check with `swagger_get_info`, load if needed)
2. If auth is required: `swagger_set_auth({ token: "Bearer xxx" })` — token is used as-is
3. `swagger_get_endpoint({ method, path })` → `Read` cache file for parameter details
4. `swagger_call_api({ method, path, parameters, body, confirmed: false })` — preview the request
5. Review the preview with the user
6. `swagger_call_api({ ..., confirmed: true })` — execute after confirmation
7. Present and explain the response

**Key rules**:
- Always preview first (`confirmed: false`), then confirm (`confirmed: true`)
- `token` in `swagger_set_auth` is the full header value (include "Bearer " prefix yourself)
- Use `parameters` for query/path params, `body` for request body

## Schema Workflow

1. `swagger_list_schemas({ keyword })` — find schemas by name
2. `swagger_get_schema({ name })` — get summary + cache file path
3. `Read` the cache file for full property definitions, nested objects, and enums
4. Explain the schema structure to the user

## Full Integration Workflow

Combine all workflows for end-to-end API integration:

1. **Load**: `swagger_load_spec` → `swagger_get_info` (understand auth requirements)
2. **Discover**: `swagger_search` or `swagger_list_paths` to find relevant endpoints
3. **Understand**: `swagger_get_endpoint` → `Read` cache file → `swagger_get_schema` for data models
4. **Auth**: `swagger_set_auth` if needed
5. **Call**: `swagger_call_api` with preview → confirm
6. **Iterate**: Use results to find next endpoints as needed

## Token Efficiency

Tools return ~200 chars (summary + file path) instead of 5-20KB of inline content. Follow this pattern:

```
swagger_get_endpoint → returns summary + "/path/to/.swagger-cache/endpoints/GET__users.json"
Read tool on that path → returns full endpoint details
```

Never skip the `Read` step — the tool summary alone lacks parameter details, request bodies, and response schemas.

## Cache Management

- Cache persists across conversations. If spec is already loaded, skip `swagger_load_spec`.
- Use `swagger_update_cache` to refresh if the spec has changed.
- Check `swagger_get_info` to verify the loaded spec matches the user's target API.
