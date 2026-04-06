# Nexus MCP Server

> Agent-native Ops OS for ecommerce and retail — CRM, orders, inventory, fulfillment, shipping, omnichannel messaging, social publishing, team chat, conversation intelligence, and more. All through a single MCP connection.

[![MCP](https://img.shields.io/badge/MCP-2026--04--06-blue)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/license-Proprietary-orange)]()

## What is Nexus?

Nexus is a multi-tenant operations platform built for SMEs. It consolidates:

- **CRM** — contacts, companies, journey stages, customer-360 views
- **Order Management** — full lifecycle from pending → delivered
- **Inventory & Warehousing** — stock tracking, bins, cycle counts
- **Fulfillment** — pick, pack, ship workflows
- **Shipping** — AWB creation, courier integration (Bosta), tracking
- **Omnichannel Messaging** — WhatsApp, Facebook Messenger, Instagram DMs
- **Social Publishing** — Facebook & Instagram drafts, scheduling, analytics, content calendar
- **Internal Team Chat** — agent ↔ human DMs inside the org
- **Organization Invites** — invite, cancel, and resend member invitations via agent
- **VoIP/PBX** — click-to-call, CDR sync, extensions
- **Conversation Intelligence** — AI message annotations, CS analytics, tag definitions, org AI profiles
- **Automation** — event-driven workflows, abandoned cart recovery

## For AI Agents

Nexus is designed for **agent-first** access. AI agents authenticate with an API key and get 40+ MCP tools covering all business operations.

### Quick Start (for agents)

```bash
# Step 1: A human admin creates an API key
curl -X POST "https://api.nexus.aiforstartups.io/functions/v1/agent-api-key-create" \
  -H "Authorization: Bearer <human-admin-jwt>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-agent",
    "scopes": ["read", "write"],
    "expires_at": "2026-12-31T23:59:59Z"
  }'
# Returns: api_key (save this — shown only once), organization_id

# Step 2: Exchange API key for a short-lived JWT
curl -X POST "https://api.nexus.aiforstartups.io/functions/v1/agent-auth" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "nxs_ak_..."}'
# Returns: access_token (JWT, 1 hour), organization_id

# Step 3: Use MCP
curl -X POST "https://api.nexus.aiforstartups.io/functions/v1/mcp-server" \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

## MCP Tools

### CRM

| Tool | Description |
|---|---|
| `nexus_list_contacts` | List/search contacts with filters |
| `nexus_get_contact` | Get full contact details |
| `nexus_create_contact` | Create a new contact |
| `nexus_update_contact` | Update contact fields |

### Orders

| Tool | Description |
|---|---|
| `nexus_list_orders` | List orders with filters |
| `nexus_get_order` | Get order with line items |
| `nexus_create_order` | Create order with line items |
| `nexus_update_order_status` | Move order through lifecycle |

### Inventory

| Tool | Description |
|---|---|
| `nexus_list_inventory` | List inventory items |
| `nexus_check_stock` | Check stock for item/SKU |

### Messaging

| Tool | Description |
|---|---|
| `nexus_list_conversations` | List omnichannel conversations |
| `nexus_send_message` | Send via WhatsApp/FB/IG |
| `nexus_search` | Global search across all data |

### Social

| Tool | Description |
|---|---|
| `nexus_sync_social_posts` | Sync posts from connected accounts |
| `nexus_list_social_posts` | List social posts |
| `nexus_get_social_post` | Get a single post |
| `nexus_create_social_post` | Create a draft post |
| `nexus_update_social_post` | Update a draft post |
| `nexus_publish_social_post` | Publish a post |
| `nexus_delete_social_post` | Delete a post |
| `nexus_list_social_accounts` | List connected social accounts |
| `nexus_get_social_analytics` | Get social analytics |
| `nexus_get_content_calendar` | Get scheduled content calendar |

### Organization

| Tool | Description |
|---|---|
| `nexus_list_organization_invites` | List pending invites |
| `nexus_invite_organization_member` | Invite a new member |
| `nexus_cancel_organization_invite` | Cancel a pending invite |
| `nexus_resend_organization_invite` | Resend an invite email |

### Internal Team Chat

| Tool | Description |
|---|---|
| `nexus_list_internal_conversations` | List agent's internal conversations |
| `nexus_get_internal_messages` | Get messages in a conversation |
| `nexus_open_internal_dm` | Open a DM with an org member |
| `nexus_send_internal_message` | Send an internal message |

### Conversation Intelligence (CS AI)

| Tool | Description |
|---|---|
| `nexus_list_ai_tag_definitions` | List AI tag taxonomy |
| `nexus_list_message_annotations` | List message annotations |
| `nexus_get_message_ai_annotation` | Get annotation for a message |
| `nexus_get_org_ai_annotation_settings` | Get org AI settings |
| `nexus_get_customer_conversation_intelligence` | Get customer intelligence profile |
| `nexus_get_conversation_intelligence_dashboard` | Get CS analytics dashboard |
| `nexus_update_message_annotation` | Update a message annotation |
| `nexus_update_ai_tag_definition` | Update a tag definition |
| `nexus_set_org_ai_annotation_prompt` | Set org AI prompt |
| `nexus_update_org_conversation_intelligence_context` | Update org AI context |

## MCP Resources

| Resource | Description |
|---|---|
| `nexus://organization/info` | Current org details |
| `nexus://schema/contacts` | Contact object schema |
| `nexus://schema/orders` | Order object schema |
| `nexus://schema/inventory` | Inventory object schema |
| `nexus://schema/social` | Social post schema |
| `nexus://schema/conversation-intelligence` | Conversation intelligence schema |

## Pricing

| Tier | Price | Contacts | Orders | Messages | MCP Scopes |
|---|---|---|---|---|---|
| Free | $0 | 50 | 25 | 0 | read |
| Starter | $99/mo | 500 | 200/mo | 1,000 | read, write |
| Growth | $199/mo | 5,000 | unlimited | 5,000 | full |
| Scale | $599/mo | unlimited | unlimited | 20,000 | full + AI suite |

> Social tools require Starter plan or above. Conversation intelligence AI suite requires Scale.

## OpenClaw Skill

Install the Nexus skill for OpenClaw agents:

```bash
npx clawhub@latest install nexus
```

Or install manually by copying [`SKILL.md`](SKILL.md) into your OpenClaw workspace skills directory.

## Documentation

- **Agent Quickstart**: [nexus-docs.aiforstartups.io/api/ai-agents-mcp](https://nexus-docs.aiforstartups.io/api/ai-agents-mcp)
- **Full API Reference**: [api-reference.md](api-reference.md)
- **Full Docs**: [nexus-docs.aiforstartups.io](https://nexus-docs.aiforstartups.io)
- **llms.txt**: [nexus-docs.aiforstartups.io/llms.txt](https://nexus-docs.aiforstartups.io/llms.txt)
- **OpenAPI Spec**: [nexus-docs.aiforstartups.io/openapi.yaml](https://nexus-docs.aiforstartups.io/openapi.yaml)
- **App**: [nexus.aiforstartups.io](https://nexus.aiforstartups.io)

## Tech Stack

- Frontend: React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui
- Backend: Supabase (PostgreSQL + Edge Functions on Deno)
- Auth: Supabase Auth (JWT) with multi-tenant RLS
- MCP: Streamable HTTP transport

## Built By

[AiForStartups](https://aiforstartups.io) — Made in Cairo, Egypt 🇪🇬
