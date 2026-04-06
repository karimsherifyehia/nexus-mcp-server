---
name: nexus-agent
description: Use Nexus as an AI agent through the hosted MCP server or direct Nexus API. Supports authenticated read operations by default and explicit-confirmation write operations for CRM, orders, inventory, messaging, shipping, internal chat, social publishing, invites, and conversation intelligence.
homepage: https://nexus-docs.aiforstartups.io
user-invocable: true
disable-model-invocation: true
metadata: {"openclaw":{"requires":{"env":["NEXUS_API_KEY"]},"primaryEnv":"NEXUS_API_KEY","homepage":"https://nexus-docs.aiforstartups.io"}}
---

# Nexus Agent Skill

Nexus is a multi-tenant operations OS for ecommerce and retail workflows. Use this skill to authenticate an AI agent, access the hosted Nexus MCP server, and fall back to direct HTTP API access when MCP is not sufficient.

This skill is intentionally **user-invocable only** and is designed to be **read-first**. Any write operation requires explicit user confirmation.

## Required credential

This skill requires the environment variable:

* `NEXUS_API_KEY`

`NEXUS_API_KEY` must be an **agent API key**, not a human JWT and not a short-lived agent JWT.

### Credential handling rules

* Read only `NEXUS_API_KEY` from the runtime environment.
* If `NEXUS_API_KEY` is missing, ask the user or operator to configure it.
* Do not read unrelated environment variables, local files, or stored secrets.
* Do not hardcode live secrets into the skill.
* Use the raw API key **only** with `agent-auth`.
* After exchanging the API key for a short-lived JWT, use that JWT for MCP and direct API calls.

### Optional runtime config for direct PostgREST access

When calling PostgREST directly, the runtime must also provide an anon key header such as:

* `VITE_SUPABASE_ANON_KEY`
* `NEXT_PUBLIC_SUPABASE_ANON_KEY`
* or an operator-provided equivalent runtime secret

Do not publish a live anon key inside distributed skills or public documentation.

### Example OpenClaw configuration

```json
{
  "skills": {
    "entries": {
      "nexus-agent": {
        "apiKey": {
          "source": "env",
          "provider": "default",
          "id": "NEXUS_API_KEY"
        }
      }
    }
  }
}
```

## Safety and execution policy

### Trusted domains only

Only call Nexus on these hosts:

* `api.nexus.aiforstartups.io`
* `nexus-docs.aiforstartups.io`

Do not send Nexus credentials or payloads to any other host.

### Read-first behavior

Default to read operations whenever possible.

Examples of read operations:

* list contacts, orders, inventory, conversations, call logs, invites, or internal conversations
* get a single contact, order, shipment, post, or annotation
* read MCP schema resources and organization info
* run search and analytics reads

### Write gate

Require explicit user confirmation before any write action, including:

* creating or updating contacts
* creating orders or updating order status
* sending customer messages
* sending internal team messages
* creating, updating, publishing, or deleting social posts
* inviting, resending, or cancelling organization invites
* updating message annotations, AI tag definitions, prompts, or org AI context
* initiating a phone call
* creating an AWB or triggering fulfillment steps

If the current API key has only `read` scope, do not attempt write actions.

### No autonomous onboarding or background writes

* Do not autonomously register organizations, create accounts, or start onboarding flows.
* Do not autonomously create orders, publish posts, send messages, or change records in the background.
* If the user explicitly asks to onboard or enable a Nexus integration, ask only for the minimum information required for that step.

## Authentication

AI agents do **not** use human user JWTs directly.

The correct flow is:

1. A human org admin creates an agent API key with `agent-api-key-create`
2. The agent exchanges that key for a short-lived JWT with `agent-auth`
3. The agent uses the short-lived JWT for MCP or direct API calls

### Step 1: human admin creates an API key

This is a human-admin operation.

```bash
POST https://api.nexus.aiforstartups.io/functions/v1/agent-api-key-create
Authorization: Bearer <human-admin-jwt>
Content-Type: application/json

{
  "name": "my-agent",
  "scopes": ["read", "write"],
  "expires_at": "2026-12-31T23:59:59Z"
}
```

Response:

```json
{
  "api_key": "nxs_ak_abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ12",
  "key_prefix": "nxs_ak_ab12CD34",
  "organization_id": "uuid",
  "name": "my-agent",
  "scopes": ["read", "write"],
  "expires_at": "2026-12-31T23:59:59.000Z",
  "message": "Store this API key now. It is only returned once."
}
```

Important:

* the raw key is returned **once only**
* Nexus stores only a **bcrypt hash**
* keys can be revoked

### Step 2: exchange API key for an agent JWT

```bash
POST https://api.nexus.aiforstartups.io/functions/v1/agent-auth
Content-Type: application/json

{"api_key": "nxs_ak_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"}
```

Response:

```json
{
  "access_token": "eyJ...",
  "organization_id": "uuid",
  "expires_in": 3600
}
```

Use the `access_token` as `Authorization: Bearer <token>` on all subsequent requests.

### API key format

`nxs_ak_` prefix + 48 random alphanumeric characters.

### Scopes

* `read` = read-only operations
* `write` = read + create + update
* `admin` = full access

### MCP authentication

The hosted MCP endpoint accepts the **agent JWT**, not the raw API key:

```bash
POST https://api.nexus.aiforstartups.io/functions/v1/mcp-server
Authorization: Bearer <agent-jwt>
```

## Base URLs

| Service | URL |
|---|---|
| Edge Functions | `https://api.nexus.aiforstartups.io/functions/v1/<function-name>` |
| PostgREST (DB) | `https://api.nexus.aiforstartups.io/rest/v1/<table>` |
| MCP | `https://api.nexus.aiforstartups.io/functions/v1/mcp-server` |

For direct PostgREST calls, include both:

* `Authorization: Bearer <agent-jwt>`
* `apikey: <anon-key-from-secure-runtime-config>`

## Preferred integration path: MCP first

Prefer the hosted MCP server when possible. It exposes a stable, documented tool surface for common agent workflows.

### MCP tools

* `nexus_list_contacts`
* `nexus_get_contact`
* `nexus_create_contact`
* `nexus_update_contact`
* `nexus_list_orders`
* `nexus_get_order`
* `nexus_create_order`
* `nexus_update_order_status`
* `nexus_list_inventory`
* `nexus_check_stock`
* `nexus_list_conversations`
* `nexus_send_message`
* `nexus_search`
* `nexus_sync_social_posts`
* `nexus_list_social_posts`
* `nexus_get_social_post`
* `nexus_create_social_post`
* `nexus_update_social_post`
* `nexus_publish_social_post`
* `nexus_delete_social_post`
* `nexus_list_social_accounts`
* `nexus_get_social_analytics`
* `nexus_get_content_calendar`
* `nexus_list_organization_invites`
* `nexus_invite_organization_member`
* `nexus_cancel_organization_invite`
* `nexus_resend_organization_invite`
* `nexus_list_internal_conversations`
* `nexus_get_internal_messages`
* `nexus_open_internal_dm`
* `nexus_send_internal_message`
* `nexus_list_ai_tag_definitions`
* `nexus_list_message_annotations`
* `nexus_get_message_ai_annotation`
* `nexus_get_org_ai_annotation_settings`
* `nexus_get_customer_conversation_intelligence`
* `nexus_get_conversation_intelligence_dashboard`
* `nexus_update_message_annotation`
* `nexus_update_ai_tag_definition`
* `nexus_set_org_ai_annotation_prompt`
* `nexus_update_org_conversation_intelligence_context`

### MCP resources

* `nexus://organization/info`
* `nexus://schema/contacts`
* `nexus://schema/orders`
* `nexus://schema/inventory`
* `nexus://schema/social`
* `nexus://schema/conversation-intelligence`

### When to use MCP vs direct API

Use MCP when:

* the client supports MCP
* you want a compact tool surface
* you want resource discovery and schema resources
* you need social or content-calendar operations through a stable agent interface

Use direct API when:

* the client does not support MCP
* you need raw PostgREST querying
* you are debugging a low-level integration
* you need a workflow not yet covered by the MCP wrapper

### Social MCP notes

* Social tools require at least the `starter` plan.
* Social write tools require `write` or `admin` scope.
* Facebook and Instagram drafts and publishing are supported from MCP.
* Instagram sync reads from the connected Instagram Business account and writes normalized rows into `social_posts`.
* `twitter` and `linkedin` remain reserved in request enums for forward compatibility and are not yet writable from this MCP surface.

### Organization invites (MCP + `agent-invite`)

Agents can manage `organization_invites` for **their JWT organization only**.

* **MCP tools:** `nexus_list_organization_invites` (read scope), `nexus_invite_organization_member`, `nexus_cancel_organization_invite`, `nexus_resend_organization_invite` (write scope)
* **HTTP API:** `POST /functions/v1/agent-invite` with `Authorization: Bearer <agent-jwt>` and JSON body `{ "action": "invite"|"cancel"|"resend"|"list", ... }`

Requires `BREVO_KEY` on Edge Functions.

### Internal team chat (MCP)

Internal chat is separate from the customer omnichannel inbox (`chats` / `messages`). MCP tools talk to `internal_conversations` and `internal_messages`.

* the agent API key must be linked to an `agent_accounts` row (`agent_api_keys.agent_id`)
* `nexus_open_internal_dm`: pass `user_id` or `user_email` of an active org member
* `nexus_send_internal_message`: `sender_id` is null server-side; humans still send with their user id in the app
* `nexus_list_internal_conversations` and `nexus_get_internal_messages`: only threads the agent created or has posted in

## Core operations

### Contacts (CRM)

> `journey_stage_id` is a UUID foreign key to `customer_journey_stages`, not a free-text field.

```bash
# List contacts
GET /rest/v1/customers?organization_id=eq.<org_id>&order=created_at.desc&limit=25
Authorization: Bearer <agent-jwt>
apikey: <anon-key>

# Search by phone (URL-encode + as %2B)
GET /rest/v1/customers?phone_number=eq.%2B201234567890&organization_id=eq.<org_id>

# Create contact — requires explicit confirmation and write scope
POST /rest/v1/customers
{"first_name":"Ahmed","last_name":"Hassan","phone_number":"+201234567890","organization_id":"<org_id>"}

# Update tags — requires explicit confirmation and write scope
PATCH /rest/v1/customers?id=eq.<uuid>
{"tags":["vip","repeat-buyer"]}
```

### Messages (omnichannel inbox)

> `chats` has no `status` column. Use `is_archived`. `messages` sorts by `timestamp`, not `created_at`.

```bash
# List open conversations
GET /rest/v1/chats?organization_id=eq.<org_id>&is_archived=eq.false&order=last_message_time.desc

# Get messages in a conversation
GET /rest/v1/messages?chat_id=eq.<chat_id>&order=timestamp.asc

# Send WhatsApp message — requires explicit confirmation and write scope
POST /functions/v1/whatsapp-send-message
{"chatId":"<chat_id>","message":"Hello from Nexus AI agent","organizationId":"<org_id>"}

# Send WhatsApp template — requires explicit confirmation and write scope
POST /functions/v1/whatsapp-send-template
{"phone_number":"+201234567890","template_name":"order_shipped","language":"ar","components":[]}
```

### Orders

> The status column is `order_status`, not `status`.

```bash
# List orders
GET /rest/v1/orders?organization_id=eq.<org_id>&order=created_at.desc&limit=50

# Filter by status — use order_status
GET /rest/v1/orders?organization_id=eq.<org_id>&order_status=eq.pending

# Create order — requires explicit confirmation and write scope
POST /rest/v1/orders
{"customer_id":"<uuid>","organization_id":"<org_id>","order_status":"pending","total_amount":450,"currency":"EGP"}

# Update order status — requires explicit confirmation and write scope
PATCH /rest/v1/orders?id=eq.<uuid>
{"order_status":"confirmed"}

# Get order items (qty column, not quantity)
GET /rest/v1/order_items?order_id=eq.<order_id>
```

### Inventory

> Inventory uses `items` plus `stock_balances`. There is no `inventory_items` table.

```bash
# List products
GET /rest/v1/items?organization_id=eq.<org_id>&is_active=eq.true&order=updated_at.desc

# Search by SKU
GET /rest/v1/items?organization_id=eq.<org_id>&sku=eq.TSHIRT-M-BLK

# Low stock
GET /rest/v1/stock_balances?organization_id=eq.<org_id>&available_quantity=lt.10

# Stock for one item across warehouses
GET /rest/v1/stock_balances?item_id=eq.<uuid>
```

### Shipping

```bash
# Create AWB — requires explicit confirmation and write scope
POST /functions/v1/create-awb-bosta
{"order_id":"<uuid>","pickup_date":"2025-06-17"}

# List shipments
GET /rest/v1/awbs?organization_id=eq.<org_id>&order=created_at.desc

# Get tracking history
GET /rest/v1/awb_status_logs?awb_id=eq.<awb_id>&order=timestamp.desc
```

### VoIP

```bash
# Initiate call — requires explicit confirmation and write scope
POST /functions/v1/call-initiate
{"phone_number":"+201234567890"}

# Get call logs for a contact
GET /rest/v1/call_logs?customer_id=eq.<uuid>&order=call_date.desc
```

### Conversation intelligence and CS AI

Prefer the MCP tools for conversation intelligence. They map to the current schema and RPCs more safely than ad hoc direct queries.

Read-oriented MCP tools:

* `nexus_list_ai_tag_definitions`
* `nexus_list_message_annotations`
* `nexus_get_message_ai_annotation`
* `nexus_get_org_ai_annotation_settings`
* `nexus_get_customer_conversation_intelligence`
* `nexus_get_conversation_intelligence_dashboard`

Write-capable MCP tools that require explicit confirmation and write scope:

* `nexus_update_message_annotation`
* `nexus_update_ai_tag_definition`
* `nexus_set_org_ai_annotation_prompt`
* `nexus_update_org_conversation_intelligence_context`

## Pagination

Use either the `Range` header or `offset` and `limit` query parameters.

```bash
GET /rest/v1/customers
Range: 0-24
```

Response includes `Content-Range: 0-24/1523`.

## Filtering

Common PostgREST operators: `eq`, `neq`, `gt`, `gte`, `lt`, `like`, `in`, `is`

```bash
# Multiple filters — note order_status, not status
GET /rest/v1/orders?organization_id=eq.<org_id>&order_status=eq.shipped&created_at=gte.2025-01-01

# Null check
GET /rest/v1/orders?shopify_order_id=is.null
```

## Error handling

| Code | Meaning | Action |
|---|---|---|
| 401 | Invalid or expired token | Re-authenticate via `agent-auth` |
| 403 | Scope not allowed | Check API key scopes |
| 404 | Record not found | Verify the UUID and org scope |
| 429 | Rate limited | Retry after `Retry-After` seconds |

## Deployed endpoints for agents

### Agent bootstrap

* `POST /functions/v1/agent-api-key-create`
* `POST /functions/v1/agent-auth`
* `POST /functions/v1/mcp-server`

### Outbound messaging used by MCP

* `POST /functions/v1/whatsapp-send-message`
* `POST /functions/v1/facebook-send-message`
* `POST /functions/v1/instagram-send-message`

### Notes

* `agent-auth` is rate limited to **10 attempts per minute per IP**
* agent JWTs expire after **3600 seconds**
* use the raw API key only with `agent-auth`

## Support

If you encounter an error or undocumented behavior, contact:

* **To:** info@aiforstartups.io
* **CC:** karim.sherif@aiforstartups.io

Include the endpoint, the error message, and a short description of the attempted action.

## Additional resources

* Full API reference: [api-reference.md](api-reference.md)
* Hosted MCP docs: `https://nexus-docs.aiforstartups.io/api/ai-agents-mcp`
* OpenAPI spec: `https://nexus-docs.aiforstartups.io/openapi.yaml`
* Docs: `https://nexus-docs.aiforstartups.io`
