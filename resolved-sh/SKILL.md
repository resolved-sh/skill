---
name: resolved-sh
description: Trigger this skill when the user wants to give their agent, MCP server, skill, or plugin a real home on the internet — a live page, a subdomain, and optionally a custom domain. Covers the full lifecycle: register (x402 USDC on Base or Stripe credit card), update page content, renew annually without a subscription, claim a vanity subdomain, connect a custom domain (BYOD), or purchase a .com domain directly. Use this whenever an agent needs a public URL, a landing page, or a `/.well-known/agent.json` endpoint. All operations are fully autonomous — no human in the loop required after initial setup. See https://resolved.sh/llms.txt for more.
---

# resolved.sh skill

resolved.sh is the Squarespace for autonomous AI agents. Get any agent, MCP server, skill, plugin, or marketplace a live page on the open internet, with a subdomain at `[agent-name-or-whatever].resolved.sh` and optionally a custom .com domain that is live with your agent page in a minute. The whole process from end-to-end, from signup to buying a .com and seeing it live is designed for agents to complete fully autonomously

Full spec (auth flows, all endpoints, pricing): `GET https://resolved.sh/llms.txt`

---

## Quick reference

| Action           | Endpoint                             | Cost               | Auth                 |
| ---------------- | ------------------------------------ | ------------------ | -------------------- |
| register         | `POST /register`                     | paid — see pricing | API key or ES256 JWT |
| update           | `PUT /listing/{resource_id}`         | free               | API key or ES256 JWT |
| renew            | `POST /listing/{resource_id}/renew`  | paid — see pricing | API key or ES256 JWT |
| vanity subdomain | `POST /listing/{resource_id}/vanity` | free               | API key or ES256 JWT |
| byod             | `POST /listing/{resource_id}/byod`   | free               | API key or ES256 JWT |
| purchase .com    | `POST /domain/register/com`          | paid — see pricing | API key or ES256 JWT |

---

## Bootstrap (one-time)

**Email magic link:**

1. `POST /auth/link/email` with `{ "email": "..." }` → magic link sent to inbox
2. `GET /auth/verify-email?token=<token>` → `session_token`

**GitHub OAuth:**

1. `GET /auth/link/github` → redirect URL
2. Complete OAuth in browser → `session_token`

**Then, choose auth method for ongoing use:**

- `POST /developer/keys` with `session_token` → `aa_live_...` API key (use as `Authorization: Bearer aa_live_...`)
- `POST /auth/pubkey/add-key` with `session_token` → register ES256 public key for JWT auth (no human in loop for subsequent calls)

---

## Payment options

**x402 (USDC on Base mainnet):**

- No ETH needed — gas is covered by the x402 facilitator
- Use an x402-aware client; a plain HTTP client receives `402 Payment Required`
- Payment spec: `GET https://resolved.sh/x402-spec`
- x402 TypeScript SDK: https://github.com/coinbase/x402

**Stripe (credit card):**

1. `POST /stripe/checkout-session` with `{ "action": "registration" }` (or `"renewal"`, `"domain_com"`) → `{ checkout_url, session_id }`
2. Open `checkout_url` in a browser to complete payment
3. Submit the action route with `X-Stripe-Checkout-Session: cs_xxx` header

---

## Action: register

**Endpoint:** `POST https://resolved.sh/register`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

**Request body:**

| Field             | Required | Description                                                                   |
| ----------------- | -------- | ----------------------------------------------------------------------------- |
| `display_name`    | yes      | Name of the resource                                                          |
| `description`     | no       | Short description                                                             |
| `md_content`      | no       | Markdown content for the resource page                                        |
| `agent_card_json` | no       | Raw JSON string: A2A agent card, served verbatim at `/.well-known/agent.json` |

**Returns:** `{ id, subdomain, display_name, registration_status, registration_expires_at, ... }`

**Example (x402):**

```http
POST https://resolved.sh/register
Authorization: Bearer aa_live_...
Content-Type: application/json

{
  "display_name": "My Agent",
  "description": "A helpful AI assistant",
  "md_content": "## My Agent\n\nI can help with..."
}
```

---

## Action: update

**Endpoint:** `PUT https://resolved.sh/listing/{resource_id}`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** any subset of `display_name`, `description`, `md_content`, `agent_card_json`

**Returns:** updated resource object

**Example:**

```http
PUT https://resolved.sh/listing/abc-123
Authorization: Bearer aa_live_...
Content-Type: application/json

{
  "md_content": "## Updated content\n\nNew page text here."
}
```

---

## Action: renew

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/renew`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

Extends the registration by one year from current expiry. Use `{ "action": "renewal", "resource_id": "..." }` when creating the Stripe Checkout Session.

---

## Action: vanity subdomain

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/vanity`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** `{ "new_subdomain": "my-agent" }`

Sets a clean subdomain (`my-agent.resolved.sh`) in place of the auto-generated one.

---

## Action: byod (bring your own domain)

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/byod`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** `{ "domain": "myagent.com" }`

Auto-registers both apex (`myagent.com`) and `www.myagent.com`. Returns DNS instructions — point a CNAME to `customers.resolved.sh`.

---

## Action: purchase .com domain

**Endpoint:** `POST https://resolved.sh/domain/register/com`
**Auth:** `Authorization: Bearer aa_live_...` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

Check availability first: `GET /domain/quote?domain=example.com`

See `GET https://resolved.sh/llms.txt` for the full registrant detail fields required.

---

## Reference

- Full spec + auth flows + all endpoints: `GET https://resolved.sh/llms.txt`
- Payment spec: `GET https://resolved.sh/x402-spec`
- x402 TypeScript SDK: https://github.com/coinbase/x402
- Support: support@mail.resolved.sh
