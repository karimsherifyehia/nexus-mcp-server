---
name: nexus
description: Interact with Nexus, a multi-tenant Ops OS for ecommerce and retail businesses. Use when the user asks about CRM contacts, orders, inventory, fulfillment, shipping, omnichannel messaging (WhatsApp, Facebook, Instagram), or business operations. Also use when asked to check stock, manage orders, send messages to customers, search contacts, or perform any ecommerce ops task. Triggers on phrases like "check inventory", "list orders", "send WhatsApp", "customer lookup", "stock level", "create order", "order status", "contact info", "CRM", "fulfillment", "shipping".
---

# Nexus — Ops OS for AI Agents

Nexus is a multi-tenant operations platform with CRM, orders, inventory, fulfillment, shipping, omnichannel messaging, VoIP, AI analytics, and automation. This skill enables interaction via the Nexus MCP server.

## Credentials

Read credentials from `{baseDir}/../../.nexus-credentials`. If not found, run the registration flow below.

## Registration (first-time setup)

If no credentials exist, self-register:

```bash
curl -s -X POST "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-register" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_name": "<agent-name>",
    "agent_platform": "openclaw",
    "owner_email": "<owner-email>",
    "organization_name": "<org-name>",
    "plan": "free"
  }'
```

Save the returned `api_key` and `organization_id` to `.nexus-credentials` in the workspace root. The API key is shown only once.

## Authentication

Exchange the API key for a short-lived JWT (1 hour):

```bash
curl -s -X POST "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-auth" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "<your-nxs_ak-key>"}'
```

Returns `access_token` (JWT) and `expires_in` (3600 seconds). Use the JWT as `Authorization: Bearer <jwt>` for all subsequent calls.

## MCP Server

Endpoint: `https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/mcp-server`

All MCP calls require:
- `Authorization: Bearer <agent-jwt>`
- `Content-Type: application/json`
- `Accept: application/json, text/event-stream`

### Initialize

```bash
curl -s -X POST "<mcp-endpoint>" \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"<agent-name>","version":"1.0.0"}}}'
```

### Call a Tool

```bash
curl -s -X POST "<mcp-endpoint>" \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"<tool-name>","arguments":{}}}'
```

## Available Tools

| Tool | Scope | Description |
|---|---|---|
| `nexus_list_contacts` | read | List/search contacts with filters (phone, email, journey_stage, search) |
| `nexus_get_contact` | read | Get full contact details + recent orders by UUID |
| `nexus_create_contact` | write | Create a new contact (first_name, last_name, email, phone, tags) |
| `nexus_update_contact` | write | Update contact fields by UUID |
| `nexus_list_orders` | read | List orders with filters (status, customer_id, date range) |
| `nexus_get_order` | read | Get order with line items by UUID |
| `nexus_create_order` | write | Create order with line items (customer_id, items, shipping) |
| `nexus_update_order_status` | write | Move order through lifecycle (pending→confirmed→fulfilling→shipped→delivered) |
| `nexus_list_inventory` | read | List inventory items (filter by SKU, category, low_stock) |
| `nexus_check_stock` | read | Check stock balance for item_id or SKU |
| `nexus_list_conversations` | read | List omnichannel conversations (whatsapp, facebook, instagram) |
| `nexus_send_message` | write | Send message via WhatsApp/Facebook/Instagram |
| `nexus_search` | read | Global search across contacts, orders, and inventory |

## Plan Limits

Free tier: read-only MCP, 50 contacts, 25 orders, 500 API calls/day, no outbound messaging.
Upgrade at https://nexus.aiforstartups.io/pricing for write access and messaging.

## Full Documentation

- Agent docs: https://nexus-docs.aiforstartups.io/api/ai-agents-mcp
- llms.txt: https://nexus-docs.aiforstartups.io/llms.txt
- OpenAPI spec: https://nexus-docs.aiforstartups.io/openapi.yaml
