# Nexus API Quick Reference

## Endpoints

| Purpose | Method | URL |
|---|---|---|
| Agent registration | POST | `https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-register` |
| API key → JWT | POST | `https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/agent-auth` |
| MCP server | POST | `https://lgwvoomgrwpsgpxwyaec.supabase.co/functions/v1/mcp-server` |

## Pricing Tiers

| Tier | Price | Contacts | Orders | Messages | API calls/day | MCP Scopes |
|---|---|---|---|---|---|---|
| Free | $0 | 50 | 25 | 0 | 500 | read |
| Starter | $99/mo | 500 | 200/mo | 1,000 | 5,000 | read, write |
| Growth | $199/mo | 5,000 | unlimited | 5,000 | 25,000 | read, write, admin |
| Scale | $599/mo | unlimited | unlimited | 20,000 | 100,000 | full + AI suite |

## Order Lifecycle

pending → confirmed → fulfilling → shipped → delivered
                                              → returned
                    → cancelled

## Conversation Channels

whatsapp, facebook, instagram, email, internal

## Auth Flow

1. Register: POST agent-register → get api_key (one-time)
2. Auth: POST agent-auth with api_key → get JWT (1 hour)
3. Use: POST mcp-server with Bearer JWT → call tools

## Error Codes

- 401: Invalid/expired JWT — re-authenticate via agent-auth
- 403: Plan limit exceeded — check agent-usage endpoint
- 429: Rate limited — wait and retry
