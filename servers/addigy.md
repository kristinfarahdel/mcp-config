# Addigy MCP Server

Apple device management (MDM) platform for IT admins and MSPs.

- **Repo:** [rinsed-org/mcp_addigy](https://github.com/rinsed-org/mcp_addigy)
- **API docs:** https://api.addigy.com/api/v2/documentation/

## API Quirks Discovered

These were found during E2E testing — not documented in the API docs:

- **Base URL is always `https://api.addigy.com`** — the web UI URL (e.g., `ae764710-...addigy.com`) is NOT the API
- **Auth is `x-api-key` header**, not `Authorization: Bearer`
- **All paths require trailing slashes** — returns 307 redirect without them
- **Device filters use `audit_field`/`operation`/`type`/`value`** — not `field`/`operator`/`value`
- **POST query endpoints need `sort_direction` in body** — some also need `sort_field` (singular) or `sort_fields` (plural array, for community)
- **Policies endpoint returns a plain array** — not `{items, metadata}` like other endpoints. Must wrap it.
- **System events `queries` field takes `[{"query": "text"}]`** — not `["text"]`
- **Org-scoped endpoints** (`/api/v2/o/{org_id}/`) required for: end users, SentinelOne, compliance CRUD, variable CRUD, software CRUD, policy items, alerts management, all write/delete ops
- **`search_any` field conflict** — serial_number and user_email both map to it, can't use both at once

## Environment Variables

```bash
ADDIGY_API_TOKEN=your-token        # Required — x-api-key header
ADDIGY_ORG_ID=your-org-uuid        # Recommended — needed for most write ops
ADDIGY_MULTITENANCY=false          # Optional — set true for MSPs with child orgs
```

## Tools (22 total)

### Read-only (11)
get_devices, get_policies, get_alerts, get_compliance, get_custom_facts, get_software, get_mdm_profiles, get_sentinelone, get_end_users, get_community, get_variables, get_system_events

### Write/delete (11)
device_action, manage_alerts, manage_compliance, manage_custom_facts, manage_software, manage_policy_items, manage_mdm_profiles, manage_sentinelone, manage_end_users, import_community_item, manage_variables
