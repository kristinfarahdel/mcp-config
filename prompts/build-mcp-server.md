# Build an MCP Server

Copy the prompt below and give it to Claude. It will walk you through everything interactively — just answer the questions and provide your API docs URL.

## The Prompt

```
I want to build an MCP (Model Context Protocol) server so an LLM can interact with a REST API. Walk me through this step by step.

## Step 1: Gather info

Ask me these questions one at a time. Don't move on until each is answered:

1. What platform/service is this for? (e.g., "Addigy — Apple device management for IT admins")
2. What is the URL to the API docs? (Swagger/OpenAPI spec, REST API reference, developer docs — whatever you have)
3. How does the API authenticate? (e.g., "API key in x-api-key header", "Bearer token", "OAuth2" — if you're not sure, say so and I'll try to find it in the docs)
4. Do you already have an API key/token I should use? What environment variable name do you want for it? (e.g., ACME_API_TOKEN)
5. Are there any scoping concepts like org IDs, workspace IDs, or tenant IDs that some endpoints require? If so, what env var name?
6. What do you want to name this project? Default is mcp_{platform_name} (e.g., mcp_stripe, mcp_addigy). They can pick any name they want.
7. Which tools do you want? Pick one:
   a) "I know what I want" — tell me the specific tools you need and I'll build just those
   b) "I know what I want, but also find more" — tell me your tools, then I'll review the API docs and suggest additional useful ones
   c) "You tell me" — I'll review the full API docs and propose a complete tool list for you to approve

## Step 2: Analyze the API docs and propose tools

Once I have the API docs URL, fetch and analyze them. Present:

- A summary of the API: base URL, auth scheme, rate limits, pagination format
- Any quirks you notice (trailing slashes required, non-standard response shapes, required query/body params that aren't obvious)

Then, based on their answer to question 7:

- **If (a):** Build exactly the tools they asked for. Don't suggest extras.
- **If (b):** Start with their requested tools, then review the API docs for additional useful endpoints and suggest them. Group suggestions by domain area so they can accept/reject by category.
- **If (c):** Review the full API docs and propose a complete tool list.

For all options, present tools grouped by domain area, split into:
  - **Read-only tools** (GET/query endpoints — safe to call anytime)
  - **Write/delete tools** (POST/PUT/PATCH/DELETE mutations)

For each tool: the name, a one-line description, and the endpoint(s) it wraps.

Ask me to approve the tool list, add/remove tools, or adjust groupings before writing any code. They can always ask for more tools later.

## Step 3: Build it

Once the tool list is approved, build the full MCP server using this architecture:

### Tech stack
- Python 3.12+, FastMCP (fastmcp>=2.0.0), httpx for async HTTP, tenacity for retry logic
- uv for package management (not pip, not poetry)
- ruff for linting and formatting
- pytest + pytest-asyncio + respx for testing
- Hatchling build backend

### Project structure

    src/{module_name}/
        __init__.py          — Version from importlib.metadata
        __main__.py          — CLI entry point: mcp.run(transport="stdio")
        server.py            — FastMCP instance, singleton client, get_client()
        client.py            — Async HTTP client (httpx), retry logic, pagination
        tools/
            __init__.py      — READ_ONLY and WRITE ToolAnnotation constants
            {domain1}.py     — One file per domain area
            {domain2}.py
            ...

### Architecture rules
1. **Singleton HTTP client** — one shared httpx.AsyncClient, initialized lazily from env vars
2. **Tool annotations** — every tool gets `ToolAnnotations(readOnlyHint=True)` or `ToolAnnotations(readOnlyHint=False, destructiveHint=True)`
3. **Retry logic** — tenacity with exponential backoff for 429 and 5xx
4. **Pagination** — tools accept page/per_page params; tool docstrings tell the LLM to check for more pages
5. **Error handling** — return error dicts from tools, not exceptions (LLMs handle structured errors better)
6. **Confirmation gates** — destructive actions require a `confirm` bool param (e.g., `confirm_wipe: bool = False`)
7. **Tool docstrings** — include valid enum values, filter options, and pagination hints so the LLM knows how to call each tool correctly
8. **Safety instructions** — embed in FastMCP server instructions: confirm before destructive ops, state counts before batch ops

### Deliverables
- All source code
- pyproject.toml with CLI entry point
- README with setup instructions, env var reference, and Claude Desktop config snippet
- Unit tests for client + tools
- Run ruff check and ruff format before finishing

## Step 4: Test

After building, walk me through testing:

1. How to add the server to Claude Desktop (or Claude Code) config
2. Test each read-only tool first via the MCP connector
3. Fix any issues found (auth, pagination, response format quirks)
4. Apply learnings from read tools to the write tools
5. Only test write tools if I explicitly ask to

## Important lessons (from building MCP servers before)

- Always fetch and read the actual API docs — don't guess at endpoints, params, or response shapes
- Every API has undocumented quirks (trailing slashes, required sort params, non-standard auth). Find them during read-only testing before building writes.
- FastMCP tools can't return bare lists — wrap array responses in a dict
- Tool docstrings ARE the LLM's documentation. Put enum values, filter formats, and pagination info directly in them.
- Start with read-only tools, get them passing E2E, then build writes. This order catches API quirks early.
- Split into PRs by feature area, not one giant PR
```
