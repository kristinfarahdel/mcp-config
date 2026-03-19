# Build an MCP Server

A reusable prompt template for building a production MCP server from any REST API. Copy the prompt below, fill in the bracketed sections, and give it to Claude.

## The Prompt

```
Build an MCP (Model Context Protocol) server for [PLATFORM NAME] — a [one-line description of the platform].

## Tech Stack

- Python 3.12+, FastMCP (fastmcp>=2.0.0), httpx for async HTTP, tenacity for retry logic
- uv for package management (not pip)
- ruff for linting and formatting
- pytest + pytest-asyncio + respx for testing
- Hatchling build backend
- Package name: [your-package-name] (e.g., acme-mcp), importable as [your_module_name] (e.g., acme_mcp)

## API Details

- Base URL: [https://api.example.com]
- Auth method: [header name and format, e.g., "x-api-key header" or "Authorization: Bearer"]
- Auth env var: [ENV_VAR_NAME] (e.g., ACME_API_TOKEN)
- API docs: [URL to API documentation / Swagger / OpenAPI spec]
- Rate limit: [e.g., "1,000 requests per 10 seconds"]
- Pagination format: [describe how the API paginates, e.g., "POST query endpoints return {items, metadata: {page, per_page, total}}"]
- Other quirks: [e.g., "requires trailing slashes", "some endpoints use org-scoped paths /o/{org_id}/"]

## Environment Variables

- [AUTH_TOKEN_VAR] (required): API key / token
- [ORG_ID_VAR] (yes/no/recommended): [description]
- [OTHER_VAR] (optional, default: [value]): [description]

## Tools to Build

Split tools into read-only (safe queries) and write/delete (mutations). Each tool is an
async function decorated with @mcp.tool(). Annotate with ToolAnnotations(readOnlyHint=True)
for reads and ToolAnnotations(readOnlyHint=False, destructiveHint=True) for writes.

### Read-only tools

- get_[resource]: [What it queries/returns] — [METHOD /path]
- get_[resource]: [What it queries/returns] — [METHOD /path]

### Write/delete tools

- manage_[resource]: [Create/update/delete description] — [METHOD /path]
- [action]_[resource]: [What it does] — [METHOD /path]

## Safety Rules

These get embedded in the FastMCP server instructions and guide LLM behavior:

- Always confirm before executing destructive actions (e.g., [list dangerous operations])
- When batch-operating on resources, state the count and ask for confirmation
- [Any domain-specific safety rules]

## Architecture Requirements

1. Project layout: src/[module_name]/ with __init__.py, __main__.py, server.py, client.py, and tools/ subdirectory
2. Singleton HTTP client: One shared httpx.AsyncClient reused across all tool calls, initialized lazily from env vars
3. Tool organization: One file per domain area in tools/. Each file imports the shared mcp instance and get_client() from server.py
4. Tool annotations: Define READ_ONLY and WRITE constants in tools/__init__.py using mcp.types.ToolAnnotations
5. Retry logic: Use tenacity with exponential backoff for 429 and 5xx responses
6. Pagination: Client should have a paginate() method. Tools accept page and per_page params so the LLM can navigate pages
7. Error handling: Return error dicts from tools (not exceptions) — LLMs handle structured errors better than stack traces
8. Confirmation gates: Destructive actions should require a confirmation parameter (e.g., confirm_delete: bool = False)
9. CLI entry point: __main__.py with --log-level flag, runs mcp.run(transport="stdio")
10. Tests: Unit tests for client (auth, pagination, retry) and integration tests for tools (mock HTTP with respx)

## Claude Desktop Configuration

{
  "mcpServers": {
    "[server-name]": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/project", "[cli-command-name]"],
      "env": {
        "[AUTH_TOKEN_VAR]": "your-token",
        "[ORG_ID_VAR]": "your-org-id"
      }
    }
  }
}
```

## Example: Filled-in Prompt (Addigy)

```
Build an MCP server for Addigy — an Apple device management (MDM) platform for IT admins
and MSPs managing macOS, iOS, iPadOS, and tvOS fleets.

Base URL: https://api.addigy.com
Auth: x-api-key header via ADDIGY_API_TOKEN env var
Docs: https://api.addigy.com/api/v2/documentation/
Rate limit: 1,000 requests per 10 seconds
Pagination: POST query endpoints return {items, metadata: {page, per_page, page_count, total}}
Quirks: Requires trailing slashes on all paths. Some endpoints are org-scoped
(/api/v2/o/{org_id}/). Some POST endpoints require sort_direction + sort_field in body.

Read tools: get_devices, get_policies, get_alerts, get_compliance, get_custom_facts,
get_software, get_mdm_profiles, get_sentinelone, get_end_users, get_community,
get_variables, get_system_events

Write tools: device_action (lock/wipe/restart/shutdown/assign), manage_alerts,
manage_compliance, manage_custom_facts, manage_software, manage_policy_items,
manage_mdm_profiles, manage_sentinelone, manage_end_users, import_community_item,
manage_variables

Safety: Confirm before wipe/lock/disable. Never wipe without explicit "confirm wipe".
State device count before batch operations.
```

## Tips

1. **Start with read-only tools.** Get them working E2E before building writes. Lessons from reads (auth quirks, pagination, required fields) save time on writes.
2. **Test via Claude Desktop MCP connector.** Add the server to your Claude Desktop config and test each tool interactively — catches issues unit tests miss.
3. **Provide API docs or Swagger URL.** If the API has OpenAPI/Swagger docs, include the URL so Claude can reference exact endpoint details.
4. **Be specific about API quirks.** Every API has them — trailing slashes, non-standard auth, weird pagination, unexpectedly named fields. Call these out explicitly.
5. **Split into PRs by feature area.** Group related tools (e.g., "device management", "alert monitoring") into separate branches and PRs.
6. **Iterate on tool descriptions.** The tool's docstring becomes its MCP schema description — this is what the LLM reads to decide when/how to use the tool. Make it actionable and specific.
7. **Include valid enum values in descriptions.** If a param accepts specific values (statuses, categories, actions), list them in the docstring so the LLM knows what to pass.
8. **Mention pagination in tool descriptions.** Tell the LLM "Results are paginated — check metadata.page_count for more pages" so it fetches additional pages when needed.
