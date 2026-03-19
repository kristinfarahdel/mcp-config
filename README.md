# mcp-config

Reusable prompts, templates, and configurations for building MCP (Model Context Protocol) servers with Claude.

## What's here

```
prompts/          Reusable prompt templates for common tasks
servers/          Per-server configs and notes (env vars, endpoints, quirks)
claude/           Claude Code settings, CLAUDE.md files, hooks
```

## Prompts

| Prompt | Description |
|--------|-------------|
| [Build an MCP Server](prompts/build-mcp-server.md) | Step-by-step template for building a production MCP server from any REST API |

## Servers

| Server | Platform | Status |
|--------|----------|--------|
| [addigy-mcp](servers/addigy.md) | Addigy Apple MDM | Active |

## Usage

Copy a prompt template, fill in the bracketed sections with your API details, and give it to Claude. See each prompt file for instructions and a worked example.
