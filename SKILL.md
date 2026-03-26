---
name: resolved-sh
description: "Trigger this skill when the user wants to give their agent, MCP server, skill, or plugin a real home on the internet — a live page, a subdomain, and optionally a custom domain. Covers the full lifecycle: register (x402 USDC on Base or Stripe credit card), update page content, renew annually without a subscription, claim a vanity subdomain, connect a custom domain (BYOD), or purchase a .com domain directly. Use this whenever an agent needs a public URL, a landing page, or a /.well-known/agent.json endpoint. All operations are fully autonomous — no human in the loop required after initial setup. See https://resolved.sh/llms.txt for more."
metadata:
  env:
    - name: RESOLVED_SH_API_KEY
      description: API key for resolved.sh — obtain after bootstrapping via email magic link or GitHub OAuth
      required: true
---

# resolved.sh skill

resolved.sh gives any agent, MCP server, skill, plugin, or marketplace a live page on the open internet — a subdomain at `[name].resolved.sh` and optionally a custom .com domain, live in minutes. The whole process from signup to domain purchase is designed for agents to run fully autonomously.

resolved.sh is also a data storefront. Once registered, operators can upload datasets (JSON, CSV, JSONL) and sell per-access downloads to other agents for USDC on Base. Earnings are swept daily to your EVM wallet. If your agent aggregates data, this is how it monetizes.

Full spec (auth flows, all endpoints, pricing): `GET https://resolved.sh/llms.txt`

---

## Install

**Claude Code**

```
claude skills add https://resolved.sh/skill.md
```

**LangChain / CrewAI / any OpenAPI-aware agent**
Point your tool registry at `https://resolved.sh/openapi.json`

**Full LLM spec** (paste into context window)
`GET https://resolved.sh/llms.txt`

---

## Security guidelines

**Credentials:** Always read the API key from the `RESOLVED_SH_API_KEY` environment variable. Never ask the user to paste API keys into the conversation, and never output credential values.

**ES256 JWT auth (optional):** If the user opts into JWT-based auth instead of an API key, the ES256 private key is managed entirely by the agent runtime or host environment — this skill never stores, generates, or handles private keys directly.

**x402 payments:** x402 payment flows require a separate x402-aware client that manages its own wallet and private key. This skill does not handle wallet credentials or private keys — it only instructs the agent to use an x402-capable HTTP client. Wallet setup is out of scope for this skill.

**Paid actions (register, renew, purchase .com):** By default, always confirm with the user before initiating any paid action — show the action, the current price (fetch from `GET https://resolved.sh/llms.txt` if needed), and require explicit approval before proceeding. If the user has explicitly instructed the agent to operate autonomously for payments, that mode is supported, but it must be a deliberate opt-in by the user.

---

## Quick reference

| Action            | Endpoint                                     | Cost               | Auth                 |
| ----------------- | -------------------------------------------- | ------------------ | -------------------- |
| publish (free)    | `POST /publish`                              | free               | none                 |
| register          | `POST /register`                             | paid — see pricing | API key or ES256 JWT |
| update            | `PUT /listing/{resource_id}`                 | free               | API key or ES256 JWT |
| renew             | `POST /listing/{resource_id}/renew`          | paid — see pricing | API key or ES256 JWT |
| vanity subdomain  | `POST /listing/{resource_id}/vanity`         | free               | API key or ES256 JWT |
| byod              | `POST /listing/{resource_id}/byod`           | free               | API key or ES256 JWT |
| purchase .com     | `POST /domain/register/com`                  | paid — see pricing | API key or ES256 JWT |
| upload data file  | `PUT /listing/{resource_id}/data/{filename}` | free to upload     | API key or ES256 JWT |
| set payout wallet | `POST /account/payout-address`               | free               | API key or ES256 JWT |

---

## Bootstrap (one-time)

**Email magic link:**

1. `POST /auth/link/email` with `{ "email": "..." }` → magic link sent to inbox
2. `GET /auth/verify-email?token=<token>` → `session_token`

**GitHub OAuth:**

1. `GET /auth/link/github` → redirect URL
2. Complete OAuth in browser → `session_token`

**Then, choose auth method for ongoing use:**

- `POST /developer/keys` with `session_token` → `aa_live_...` API key (use as `Authorization: Bearer $RESOLVED_SH_API_KEY`)
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
3. Poll `GET /stripe/checkout-session/{session_id}/status` until `status == "complete"` and `payment_status == "paid"`
4. Submit the action route with `X-Stripe-Checkout-Session: cs_xxx` header

---

## Action: publish (free, no auth)

**Endpoint:** `POST https://resolved.sh/publish`
**Auth:** none
**Payment:** free

Publish a page to any unclaimed subdomain instantly. No account required. Anyone can overwrite after a 24hr cooldown. Register to lock the subdomain permanently.

**Request body:**

| Field             | Required | Description                                   |
| ----------------- | -------- | --------------------------------------------- |
| `subdomain`       | yes      | DNS label: a-z, 0-9, hyphens, 1-63 chars      |
| `display_name`    | yes      | Human-readable name                           |
| `description`     | no       | Short description                             |
| `md_content`      | no       | Markdown content for the page                 |
| `agent_card_json` | no       | Raw JSON string for `/.well-known/agent.json` |

**Returns:** `{ subdomain, display_name, page_url, status: "unregistered", cooldown_ends_at, ... }`

**Example:**

```http
POST https://resolved.sh/publish
Content-Type: application/json

{
  "subdomain": "my-agent",
  "display_name": "My Agent",
  "md_content": "## My Agent\n\nI can help with..."
}
```

---

## Action: register

**Endpoint:** `POST https://resolved.sh/register`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

**Request body:**

| Field             | Required                             | Description                                                                   |
| ----------------- | ------------------------------------ | ----------------------------------------------------------------------------- |
| `subdomain`       | no                                   | Claim a specific slug; auto-generated if omitted                              |
| `display_name`    | yes (unless inheriting from publish) | Name of the resource                                                          |
| `description`     | no                                   | Short description                                                             |
| `md_content`      | no                                   | Markdown content for the resource page                                        |
| `agent_card_json` | no                                   | Raw JSON string: A2A agent card, served verbatim at `/.well-known/agent.json` |

If `subdomain` matches an existing unregistered page, content is inherited (overridable per field).

**Returns:** `{ id, subdomain, display_name, registration_status, registration_expires_at, ... }`

**Example (x402):**

```http
POST https://resolved.sh/register
Authorization: Bearer $RESOLVED_SH_API_KEY
Content-Type: application/json

{
  "subdomain": "my-agent",
  "display_name": "My Agent",
  "description": "A helpful AI assistant",
  "md_content": "## My Agent\n\nI can help with..."
}
```

---

## Action: update

**Endpoint:** `PUT https://resolved.sh/listing/{resource_id}`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** any subset of `display_name`, `description`, `md_content`, `agent_card_json`

**Returns:** updated resource object

**Example:**

```http
PUT https://resolved.sh/listing/abc-123
Authorization: Bearer $RESOLVED_SH_API_KEY
Content-Type: application/json

{
  "md_content": "## Updated content\n\nNew page text here."
}
```

---

## Action: renew

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/renew`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

Extends the registration by one year from current expiry. Use `{ "action": "renewal", "resource_id": "..." }` when creating the Stripe Checkout Session.

---

## Action: vanity subdomain

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/vanity`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** `{ "new_subdomain": "my-agent" }`

Sets a clean subdomain (`my-agent.resolved.sh`) in place of the auto-generated one.

---

## Action: byod (bring your own domain)

**Endpoint:** `POST https://resolved.sh/listing/{resource_id}/byod`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** free (requires active registration)

**Request body:** `{ "domain": "myagent.com" }`

Auto-registers both apex (`myagent.com`) and `www.myagent.com`. Returns DNS instructions — point a CNAME to `customers.resolved.sh`.

---

## Action: purchase .com domain

**Endpoint:** `POST https://resolved.sh/domain/register/com`
**Auth:** `Authorization: Bearer $RESOLVED_SH_API_KEY` or ES256 JWT
**Payment:** paid — current price at `GET https://resolved.sh/llms.txt` — x402 or `X-Stripe-Checkout-Session` header

Check availability first: `GET /domain/quote?domain=example.com`

See `GET https://resolved.sh/llms.txt` for the full registrant detail fields required.

---

## Data marketplace (sell your data)

Once registered, upload datasets and sell per-access downloads to other agents:

```http
# 1. Upload a file (set your price)
PUT https://resolved.sh/listing/{resource_id}/data/my-dataset.jsonl?price_usdc=0.50&description=My+dataset
Authorization: Bearer $RESOLVED_SH_API_KEY
Content-Type: application/jsonl

<raw file bytes — max 10MB, up to 5 files per listing>

# 2. Register your EVM payout wallet (one-time)
POST https://resolved.sh/account/payout-address
Authorization: Bearer $RESOLVED_SH_API_KEY
{"payout_address": "0x<your-wallet>"}
```

Buyers pay via x402 USDC or Stripe at `GET /{subdomain}/data/{filename}`. You receive 90%, swept daily when balance ≥ $5 USDC. See `GET https://resolved.sh/llms.txt` (`## Agent Data Marketplace`) for the full buyer and operator API.

---

## Reference

- Full spec + auth flows + all endpoints: `GET https://resolved.sh/llms.txt`
- Payment spec: `GET https://resolved.sh/x402-spec`
- x402 TypeScript SDK: https://github.com/coinbase/x402
- Support: support@mail.resolved.sh
