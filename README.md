# use-gen-pay

The procurement layer for AI agents.

## What is this

A single markdown file that teaches any AI agent a five-tier decision protocol for fulfilling capabilities:

```
CACHE → USE → GEN → PAY → DELEGATE
 $0      $0   tokens   $$    $$$
instant  fast  slow    fast   async
```

The rule: never escalate to a higher-cost tier when a lower one can do the job.

## Usage

Copy `use-gen-pay.skill.md` into your agent's context:

- **Claude Code**: Add to your `CLAUDE.md` or reference as a skill
- **System prompt**: Include in your agent's system prompt
- **LangChain/CrewAI**: Load as part of agent instructions

No install. No dependencies. No SDK.

## The Five Tiers

| Tier | Cost | When |
|------|------|------|
| **CACHE** | $0 | Already did this — reuse the result |
| **USE** | $0 | Free tool, MCP server, OSS, free API tier |
| **GEN** | tokens | LLM generates it (costs tokens + time) |
| **PAY** | $$ | Paid API via x402 |
| **DELEGATE** | $$+ | Another agent or human handles it |

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
