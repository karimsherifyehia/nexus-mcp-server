# Cursor Prompt: Agent Self-Registration + Pricing Tiers

## Paste everything below into Cursor:

---

I need to implement agent self-registration and a pricing/plan enforcement system for Nexus. This has three parts:

## Part 1: Plans Table and Enforcement

### Create a `plans` table:

```sql
CREATE TABLE plans (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  slug text UNIQUE NOT NULL,           -- 'free', 'starter', 'growth', 'scale'
  name text NOT NULL,
  price_usd integer NOT NULL DEFAULT 0, -- monthly price in cents (0 = free)
  limits jsonb NOT NULL,
  features jsonb NOT NULL DEFAULT '[]',
  is_active boolean DEFAULT true,
  sort_order integer DEFAULT 0,
  created_at timestamptz DEFAULT now()
);
```

Insert these plans:

**Free ($0/month):**
```json
{
  "slug": "free",
  "name": "Free",
  "price_usd": 0,
  "sort_order": 0,
  "limits": {
    "contacts": 50,
    "orders": 25,
    "inventory_items": 30,
    "outbound_messages": 0,
    "conversations_read": 20,
    "api_calls_per_day": 500,
    "warehouses": 1,
    "ai_annotations": 0,
    "agents": 1,
    "mcp_scopes": ["read"],
    "data_retention_days": 30
  },
  "features": ["crm_basic", "orders_basic", "inventory_basic", "mcp_read_only"]
}
```

**Starter ($99/month):**
```json
{
  "slug": "starter",
  "name": "Starter",
  "price_usd": 9900,
  "sort_order": 1,
  "limits": {
    "contacts": 500,
    "orders_per_month": 200,
    "inventory_items": 500,
    "outbound_messages": 1000,
    "conversations_read": -1,
    "api_calls_per_day": 5000,
    "warehouses": 1,
    "ai_annotations": 0,
    "agents": 2,
    "mcp_scopes": ["read", "write"],
    "data_retention_days": -1
  },
  "features": ["crm_full", "orders_full", "inventory_full", "fulfillment", "shipping", "messaging_whatsapp", "messaging_facebook", "messaging_instagram", "mcp_read_write", "shopify_sync"]
}
```

Note: `-1` means unlimited for that resource.

**Growth ($199/month):**
```json
{
  "slug": "growth",
  "name": "Growth",
  "price_usd": 19900,
  "sort_order": 2,
  "limits": {
    "contacts": 5000,
    "orders_per_month": -1,
    "inventory_items": -1,
    "outbound_messages": 5000,
    "conversations_read": -1,
    "api_calls_per_day": 25000,
    "warehouses": -1,
    "ai_annotations": 0,
    "agents": 5,
    "mcp_scopes": ["read", "write", "admin"],
    "data_retention_days": -1
  },
  "features": ["crm_full", "orders_full", "inventory_full", "fulfillment", "shipping", "messaging_whatsapp", "messaging_facebook", "messaging_instagram", "mcp_full", "shopify_sync", "courier_integrations", "voip_pbx", "automation", "competition_intel", "priority_support"]
}
```

**Scale ($599/month):**
```json
{
  "slug": "scale",
  "name": "Scale",
  "price_usd": 59900,
  "sort_order": 3,
  "limits": {
    "contacts": -1,
    "orders_per_month": -1,
    "inventory_items": -1,
    "outbound_messages": 20000,
    "conversations_read": -1,
    "api_calls_per_day": 100000,
    "warehouses": -1,
    "ai_annotations": -1,
    "agents": -1,
    "mcp_scopes": ["read", "write", "admin"],
    "data_retention_days": -1
  },
  "features": ["crm_full", "orders_full", "inventory_full", "fulfillment", "shipping", "messaging_whatsapp", "messaging_facebook", "messaging_instagram", "mcp_full", "shopify_sync", "courier_integrations", "voip_pbx", "automation", "competition_intel", "ai_annotations", "ai_sentiment", "ai_transcription", "ai_embeddings", "ai_cs_dashboards", "custom_integrations", "dedicated_onboarding", "sla", "priority_support"]
}
```

### Add plan reference to organizations table:

Add these columns to the `organizations` table (if they don't already exist):
- `plan_slug text REFERENCES plans(slug) DEFAULT 'free'`
- `is_agent_owned boolean DEFAULT false`
- `agent_owner_email text`
- `usage_reset_at timestamptz DEFAULT now()` — resets monthly counters

### Create a `usage_tracking` table:

```sql
CREATE TABLE usage_tracking (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid REFERENCES organizations(id) NOT NULL,
  resource text NOT NULL,              -- 'contacts', 'orders', 'outbound_messages', 'api_calls', etc.
  period text NOT NULL,                -- 'total' for lifetime counts, '2026-03' for monthly
  count integer DEFAULT 0,
  updated_at timestamptz DEFAULT now(),
  UNIQUE(organization_id, resource, period)
);
```

Add RLS to both new tables: organizations can only see their own data.

### Create a helper function `check_plan_limit`:

Create a PostgreSQL function or Edge Function called `check-plan-limit` that:
- Takes: organization_id, resource_name, increment_by (default 1)
- Looks up the org's plan limits
- Looks up current usage from usage_tracking
- Returns: `{ allowed: true/false, current: 45, limit: 50, remaining: 5 }`
- If limit is -1 (unlimited), always returns allowed: true

This function should be called by other edge functions before creating contacts, orders, sending messages, etc.

---

## Part 2: Agent Self-Registration Endpoint

Create a new Edge Function `agent-register`:

**POST, public (no auth required), heavily rate-limited.**

Request body:
```json
{
  "agent_name": "Bahig",
  "agent_platform": "openclaw",
  "owner_email": "karim@example.com",
  "organization_name": "Bahig - Personal Assistant",
  "plan": "free"
}
```

Validation:
- `agent_name`: required, string, 1-50 chars, sanitized (strip HTML/scripts)
- `agent_platform`: required, one of: "openclaw", "cursor", "claude", "chatgpt", "custom"
- `owner_email`: required, valid email format
- `organization_name`: required, string, 1-100 chars, sanitized
- `plan`: optional, defaults to "free". Only "free" is allowed for self-registration (paid plans require human upgrade)

Rate limiting:
- Max 3 registrations per owner_email per day
- Max 10 registrations per IP per hour
- Return 429 with Retry-After header when exceeded

What the function does:
1. Validate all inputs
2. Check rate limits
3. Create a Supabase Auth user:
   - email: `agent_{random_8_chars}@agents.nexus.aiforstartups.io`
   - password: random 64-character string (agent never uses password auth)
   - user_metadata: `{ type: "agent", agent_name, agent_platform, owner_email }`
   - Use the Supabase Admin API (service_role) to create the user
4. Create a new organization:
   - name: organization_name
   - plan_slug: "free"
   - is_agent_owned: true
   - agent_owner_email: owner_email
5. Assign the user to the organization as owner (update user's app_metadata with organization_id and org_role: "owner")
6. Initialize usage_tracking rows for the org (all counters at 0)
7. Generate an API key using the same logic as `agent-api-key-create`:
   - name: `"{agent_name} auto-generated key"`
   - scopes: determined by the plan's mcp_scopes (for free: `["read"]`)
   - expires_at: 90 days from now
   - Store bcrypt hash, return raw key once
8. Log in `agent_registrations` audit table

Create the `agent_registrations` audit table:
```sql
CREATE TABLE agent_registrations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_name text NOT NULL,
  agent_platform text NOT NULL,
  owner_email text NOT NULL,
  organization_id uuid REFERENCES organizations(id),
  user_id uuid,
  ip_address text,
  user_agent text,
  success boolean DEFAULT true,
  failure_reason text,
  created_at timestamptz DEFAULT now()
);
```

No RLS on agent_registrations (admin-only access, not exposed to regular users).

Response (success):
```json
{
  "success": true,
  "organization_id": "uuid",
  "api_key": "nxs_ak_...",
  "api_key_expires_at": "2026-06-23T01:00:00Z",
  "mcp_endpoint": "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/mcp-server",
  "auth_endpoint": "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-auth",
  "plan": "free",
  "plan_limits": {
    "contacts": 50,
    "orders": 25,
    "inventory_items": 30,
    "outbound_messages": 0,
    "api_calls_per_day": 500,
    "warehouses": 1,
    "ai_annotations": 0,
    "agents": 1,
    "mcp_scopes": ["read"],
    "data_retention_days": 30
  },
  "next_steps": {
    "1_auth": "POST api_key to auth_endpoint to get a 1-hour JWT",
    "2_mcp": "Use the JWT as Bearer token with the mcp_endpoint",
    "3_upgrade": "Contact sales@aiforstartups.io to upgrade your plan"
  },
  "message": "Store your API key securely. It cannot be retrieved again."
}
```

Response (rate limited):
```json
{
  "success": false,
  "error": "rate_limited",
  "message": "Registration limit exceeded. Try again later.",
  "retry_after": 3600
}
```

Response (validation error):
```json
{
  "success": false,
  "error": "validation_error",
  "message": "owner_email is required and must be a valid email address."
}
```

Security:
- NEVER expose the service_role key in any response
- Sanitize all text inputs (no HTML, no script tags, trim whitespace)
- Use CORS headers: allow all origins (agents call from anywhere)
- Log every registration attempt (success and failure) with IP and user-agent
- The generated auth user email domain (@agents.nexus.aiforstartups.io) should be clearly distinguishable from real human user emails

---

## Part 3: Agent Usage Endpoint

Create Edge Function `agent-usage`:

**GET, Bearer JWT auth (agent or human)**

Response:
```json
{
  "organization_id": "uuid",
  "plan": "free",
  "plan_name": "Free",
  "usage": {
    "contacts": { "used": 12, "limit": 50, "remaining": 38, "percentage": 24 },
    "orders": { "used": 3, "limit": 25, "remaining": 22, "percentage": 12 },
    "inventory_items": { "used": 0, "limit": 30, "remaining": 30, "percentage": 0 },
    "outbound_messages": { "used": 0, "limit": 0, "remaining": 0, "percentage": 100 },
    "api_calls_today": { "used": 47, "limit": 500, "remaining": 453, "percentage": 9 },
    "warehouses": { "used": 0, "limit": 1, "remaining": 1, "percentage": 0 },
    "agents": { "used": 1, "limit": 1, "remaining": 0, "percentage": 100 }
  },
  "upgrade_url": "https://nexus.aiforstartups.io/pricing",
  "warnings": [
    "outbound_messages: your plan does not include outbound messaging. Upgrade to Starter ($99/mo) to unlock.",
    "agents: you have reached your agent limit. Upgrade to Starter ($99/mo) for up to 2 agents."
  ]
}
```

The warnings array should automatically flag any resource at 0 remaining or above 80% usage, with a human-readable upgrade suggestion.

---

## Part 4: Update Existing Endpoints for Plan Enforcement

Update these existing edge functions to check plan limits before performing operations:

1. **Any function that creates a contact** → check `contacts` limit
2. **Any function that creates an order** → check `orders_per_month` limit (monthly reset)
3. **Any function that sends an outbound message** (whatsapp-send-message, facebook-send-message, instagram-send-message) → check `outbound_messages` limit (monthly reset)
4. **MCP server** → check `api_calls_per_day` limit on every request, and enforce `mcp_scopes` from the plan (not just from the API key — the plan is the ceiling)
5. **agent-api-key-create** → check `agents` limit before creating a new key

When a limit is exceeded, return:
```json
{
  "error": "plan_limit_exceeded",
  "resource": "outbound_messages",
  "current": 1000,
  "limit": 1000,
  "plan": "starter",
  "upgrade_message": "You've reached your monthly message limit. Upgrade to Growth ($199/mo) for 5,000 messages.",
  "upgrade_url": "https://nexus.aiforstartups.io/pricing"
}
```

Use HTTP 403 for plan limit errors (not 429, which is for rate limiting).

---

## Part 5: Update llms.txt and Documentation

Add this section to the existing llms.txt file, in the "Deployed agent endpoints" section:

```
- Agent self-registration: https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-register
- Agent usage check: https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-usage
```

Add a pricing section to llms.txt:

```
## Pricing
- Free ($0): read-only MCP, 50 contacts, 25 orders, no messaging, 500 API calls/day, 30-day data retention
- Starter ($99/mo): read+write MCP, 500 contacts, 200 orders/mo, 1K messages, 5K API calls/day
- Growth ($199/mo): full MCP, 5K contacts, unlimited orders, 5K messages, 25K API calls/day, VoIP, automation
- Scale ($599/mo): everything unlimited + AI suite (annotations, sentiment, transcription, embeddings, dashboards), custom integrations, SLA
- Additional agents: $10/agent/mo (Starter), $5/agent/mo (Growth/Scale)
```

Update the AI Agents docs page (api/ai-agents-mcp) to include the self-registration flow as "Step 0" before the existing Step 1.
