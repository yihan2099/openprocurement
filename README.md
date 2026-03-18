# openprocurement

The open procurement layer for AI agents.

## What is this

A decision protocol that teaches any AI agent how to fulfill capabilities at the lowest cost, powered by community intelligence.

```
CACHE → USE → GEN → PAY → DELEGATE
 $0      $0   tokens   $$    $$$
instant  fast  slow    fast   async
```

The rule: never escalate to a higher-cost tier when a lower one can do the job.

**Community intelligence**: every tier is informed by shared decision records from other users. Before deciding, check what others have tried and what worked.

## Install as Claude Code Plugin

```
/plugin marketplace add yihan2099/use-gen-pay
/plugin install openprocurement@openprocurement
```

Then use:
```
/openprocurement:procurement
```

Claude will automatically apply the five-tier decision protocol when fulfilling capabilities.

## Alternative Usage

You can also use the skill directly without the plugin system:

- **System prompt**: Include `skills/procurement/SKILL.md` in your agent's system prompt
- **LangChain/CrewAI**: Load as part of agent instructions

No dependencies. No SDK.

## The Five Tiers

| Tier | Cost | When |
|------|------|------|
| **CACHE** | $0 | Already did this — reuse the result |
| **USE** | $0 | Free tool, MCP server, OSS, free API tier |
| **GEN** | tokens | LLM generates it (costs tokens + time) |
| **PAY** | $$ | Paid API via x402 |
| **DELEGATE** | $$+ | Another agent or human handles it |

## Community Decisions

The `decisions/` directory contains community-submitted decision records. These are real-world results from agents and users who have run the cascade.

**To contribute:** submit a PR adding a `.yml` file to `decisions/` using the schema in `decisions/_schema.yml`.

**How it helps:**
- USE tier: see which free tools actually work best
- GEN tier: know when generating is good enough or don't bother
- PAY tier: find which providers give best value
- DELEGATE tier: learn when delegation was necessary

## Example

Agent needs to extract data from a PDF:

| Tier | Option | Cost | Quality |
|------|--------|------|---------|
| CACHE | Already extracted this file earlier | $0 | 1.0 |
| USE | Built-in PDF reader | $0 | 0.75 |
| GEN | Read + structure with LLM reasoning | tokens | 0.80 |
| PAY | DocumentAI API via x402 | $0.10 | 0.98 |
| DELEGATE | Send to specialized doc-processing agent | $0.50 | 0.99 |

With `strategy: cheapest` → checks cache first, then uses built-in PDF reader. $0 spent.

With `strategy: best_quality` → pays DocumentAI or delegates. Real money spent.

## License

MIT
