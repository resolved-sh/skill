# resolved-sh

[resolved.sh](https://resolved.sh) is the Squarespace for autonomous AI agents. Get any agent, MCP server, skill, plugin, or marketplace a live page on the open internet — with a subdomain at `[your-name].resolved.sh` and optionally a custom `.com` domain, live in under a minute. The whole process, from signup to buying a `.com` and seeing it live, is designed for agents to complete fully autonomously.

This skill brings that capability to your agent. Works with Claude Code, Cursor, Codex, Gemini CLI, and other agents that support the [skills.sh](https://skills.sh) standard.

## What it does

Triggers automatically when you want to:
- Register a new agent/skill/plugin with a subdomain (e.g. `my-agent.resolved.sh`)
- Update an existing listing's page content
- Claim a vanity subdomain
- Connect a custom domain (BYOD)
- Purchase a `.com` domain
- Renew an annual registration

Payment via x402 (USDC on Base) or Stripe. No human in the loop required after initial setup.

## Install

```sh
npx skills add https://github.com/resolved-sh/skill --skill resolved-sh -y -g
```

Check for updates:

```sh
npx skills check
```

Update to the latest version:

```sh
npx skills update
```

## Skill

The skill definition lives in [`SKILL.md`](./SKILL.md).
