# Cursor Prompt: Add .well-known/mcp.json to Nexus

Add a `.well-known/mcp.json` file to the Nexus app (nexus.aiforstartups.io) that serves as the standard MCP discovery endpoint. This is how AI agents automatically discover MCP servers on a domain.

The file should be served as static JSON at: `https://nexus.aiforstartups.io/.well-known/mcp.json`

Since Nexus is a Vite/React SPA, add this as a static file in the `public/` directory at `public/.well-known/mcp.json`.

Content:

```json
{
  "mcp_version": "2025-03-26",
  "server": {
    "name": "nexus-mcp-server",
    "version": "1.0.0",
    "description": "Nexus Ops OS — CRM, orders, inventory, fulfillment, shipping, omnichannel messaging, AI analytics for ecommerce and retail businesses.",
    "vendor": {
      "name": "AiForStartups",
      "url": "https://aiforstartups.io"
    }
  },
  "endpoints": {
    "mcp": "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/mcp-server",
    "auth": "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-auth",
    "register": "https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-register"
  },
  "transport": "streamable-http",
  "authentication": {
    "type": "bearer",
    "description": "Register via the register endpoint to get an API key, then exchange it at the auth endpoint for a short-lived JWT. Use the JWT as Bearer token.",
    "docs": "https://nexus-docs.aiforstartups.io/api/ai-agents-mcp"
  },
  "capabilities": {
    "tools": true,
    "resources": true,
    "logging": true
  },
  "documentation": {
    "llms_txt": "https://nexus-docs.aiforstartups.io/llms.txt",
    "llms_full_txt": "https://nexus-docs.aiforstartups.io/llms-full.txt",
    "openapi": "https://nexus-docs.aiforstartups.io/openapi.yaml",
    "agent_docs": "https://nexus-docs.aiforstartups.io/api/ai-agents-mcp"
  },
  "pricing": {
    "free_tier": true,
    "url": "https://nexus.aiforstartups.io/pricing",
    "plans": ["free", "starter", "growth", "scale"]
  }
}
```

Make sure the Vite config doesn't interfere with serving files from `.well-known/`. If needed, add a rewrite rule or configure the hosting to serve this path correctly.

Test: `curl -s https://nexus.aiforstartups.io/.well-known/mcp.json` should return the JSON above.
