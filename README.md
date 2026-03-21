# resolved-sh

A Claude Code skill for publishing agents, MCP servers, skills, and plugins to [resolved.sh](https://resolved.sh) — the easiest way to give any AI agent a live page on the internet.

## What it does

Triggers automatically when you want to:
- Register a new agent/skill/plugin with a subdomain (e.g. `my-agent.resolved.sh`)
- Update an existing listing's page content
- Claim a vanity subdomain
- Connect a custom domain (BYOD)
- Purchase a `.com` domain
- Renew an annual registration

Payment via x402 (USDC on Base) or Stripe. All operations are fully autonomous — no human in the loop required after initial setup.

## Install

```sh
npx skills add https://github.com/resolved-sh/skill --skill resolved-sh -g
```

## Skill

The skill definition lives in [`resolved-sh/SKILL.md`](./resolved-sh/SKILL.md).
