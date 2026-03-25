# Nexus MCP Server

> Agent-native Ops OS for ecommerce and retail — CRM, orders, inventory, fulfillment, shipping, omnichannel messaging, and AI analytics. All through a single MCP connection.

[![MCP](https://img.shields.io/badge/MCP-2025--03--26-blue)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/license-Proprietary-orange)]()

## What is Nexus?

Nexus is a multi-tenant operations platform built for SMEs. It consolidates:

- **CRM** — contacts, companies, journey stages, customer-360 views
- **Order Management** — full lifecycle from pending → delivered
- **Inventory & Warehousing** — stock tracking, bins, cycle counts
- **Fulfillment** — pick, pack, ship workflows
- **Shipping** — AWB creation, courier integration (Bosta), tracking
- **Omnichannel Messaging** — WhatsApp, Facebook Messenger, Instagram DMs
- **VoIP/PBX** — click-to-call, CDR sync, extensions
- **AI Analytics** — sentiment analysis, transcription, embeddings
- **Automation** — event-driven workflows, abandoned cart recovery

## For AI Agents

Nexus is designed for **agent-first** access. AI agents can:

1. **Discover** Nexus via [`llms.txt`](https://nexus-docs.aiforstartups.io/llms.txt) or `.well-known/mcp.json`
2. **Self-register** — no browser, no human in the loop
3. **Authenticate** — API key → short-lived JWT
4. **Operate** — 13 MCP tools covering all business operations

### Quick Start (for agents)

```bash
# Step 1: Register
curl -X POST "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-register" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_name": "my-agent",
    "agent_platform": "openclaw",
    "owner_email": "you@example.com",
    "organization_name": "My Business"
  }'
# Returns: api_key (save this — shown only once), organization_id, mcp_endpoint

# Step 2: Authenticate
curl -X POST "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-auth" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "nxs_ak_..."}'
# Returns: access_token (JWT, 1 hour), organization_id

# Step 3: Use MCP
curl -X POST "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/mcp-server" \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

## MCP Tools

| Tool | Description |
|---|---|
| `nexus_list_contacts` | List/search contacts with filters |
| `nexus_get_contact` | Get full contact details |
| `nexus_create_contact` | Create a new contact |
| `nexus_update_contact` | Update contact fields |
| `nexus_list_orders` | List orders with filters |
| `nexus_get_order` | Get order with line items |
| `nexus_create_order` | Create order with line items |
| `nexus_update_order_status` | Move order through lifecycle |
| `nexus_list_inventory` | List inventory items |
| `nexus_check_stock` | Check stock for item/SKU |
| `nexus_list_conversations` | List omnichannel conversations |
| `nexus_send_message` | Send via WhatsApp/FB/IG |
| `nexus_search` | Global search across all data |

## MCP Resources

| Resource | Description |
|---|---|
| `nexus://organization/info` | Current org details |
| `nexus://schema/contacts` | Contact object schema |
| `nexus://schema/orders` | Order object schema |
| `nexus://schema/inventory` | Inventory object schema |

## Pricing

| Tier | Price | Contacts | Orders | Messages | MCP Scopes |
|---|---|---|---|---|---|
| Free | $0 | 50 | 25 | 0 | read |
| Starter | $99/mo | 500 | 200/mo | 1,000 | read, write |
| Growth | $199/mo | 5,000 | unlimited | 5,000 | full |
| Scale | $599/mo | unlimited | unlimited | 20,000 | full + AI suite |

## Documentation

- **Agent Quickstart**: [nexus-docs.aiforstartups.io/api/ai-agents-mcp](https://nexus-docs.aiforstartups.io/api/ai-agents-mcp)
- **Full Docs**: [nexus-docs.aiforstartups.io](https://nexus-docs.aiforstartups.io)
- **llms.txt**: [nexus-docs.aiforstartups.io/llms.txt](https://nexus-docs.aiforstartups.io/llms.txt)
- **OpenAPI Spec**: [nexus-docs.aiforstartups.io/openapi.yaml](https://nexus-docs.aiforstartups.io/openapi.yaml)
- **App**: [nexus.aiforstartups.io](https://nexus.aiforstartups.io)

## OpenClaw Skill

Install the Nexus skill for OpenClaw agents:

```bash
# Coming soon to ClawHub
npx clawhub@latest install nexus
```

## Tech Stack

- Frontend: React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui
- Backend: Supabase (PostgreSQL + 117+ Edge Functions on Deno)
- Auth: Supabase Auth (JWT) with multi-tenant RLS
- MCP: Streamable HTTP transport

## Built By

[AiForStartups](https://aiforstartups.io) — Made in Cairo, Egypt 🇪🇬
