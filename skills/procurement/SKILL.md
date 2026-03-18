---
name: procurement
description: Five-tier procurement decision protocol (Cache → Use → Gen → Pay → Delegate) for fulfilling any capability at minimum cost. Includes budget tracking, caching, and community decision records.
---

# OpenProcurement — Agent Procurement Skill

You have access to this skill to decide how to fulfill any capability. Before performing any capability listed below, you MUST follow the five-tier decision protocol.

## The Five Tiers

Every capability can be fulfilled in five ways, in priority order:

1. **CACHE** ($0, instant) — You already did this. Reuse a previous result from this session. Always check first.
2. **USE** ($0, fast) — Use a free resource: MCP tool, OSS library, free API tier, local CLI, built-in tool. No cost beyond what's already running.
3. **GEN** (token cost, variable) — Generate it yourself using LLM reasoning, code generation, or multi-step tool chains. Costs tokens and time, but no external payment.
4. **PAY** (real money, fast) — Call a paid external API via x402. Highest quality and speed, but costs real money from the budget.
5. **DELEGATE** ($$+, async) — Hand it to another agent, service, or human. For tasks beyond your capabilities or that require specialized expertise, permissions, or long-running processes.

**The rule: never escalate to a higher-cost tier when a lower one can do the job.**

## Decision Protocol

When you need to perform a capability:

```
0. COMMUNITY — Check community decision records for this capability.
   → If records exist: use them to rank options at each tier below.
   → Community data tells you which USE tools work best, whether GEN is
     good enough, and which PAY providers give best value.

1. CACHE — Have I done this exact (or very similar) request already this session?
   → Yes: return the cached result. Done.

2. USE — Is there a free tool, MCP server, CLI, or free API that can do this?
   → Yes, and quality is sufficient: use it. Done.
   → Community hint: check if others report this tool works well for your context.

3. GEN — Can I generate this myself with acceptable quality?
   → Yes, and quality is sufficient: generate it. Done.
   → Community hint: check if others found GEN good enough or recommend skipping it.

4. PAY — Is there a paid API, and is it within budget?
   → Yes: pay for it via x402. Done.
   → Community hint: check which provider others recommend for best value.

5. DELEGATE — Is there another agent, service, or human who can handle this?
   → Yes: delegate with clear instructions. Done.
   → No: report to the user that this capability cannot be fulfilled.
```

After every decision, update the spend log and cache.

**Post your decision:** after each cascade run, submit your decision as a record to the community. This helps everyone make better decisions. See `decisions/_schema.yml` for the format. Submit as a PR to the openprocurement repo.

**Strategy overrides:**
- `cheapest`: Strictly follow the tier order. Only escalate if the current tier literally cannot do the job.
- `fastest`: Pick the lowest latency option across all tiers, regardless of cost.
- `best_quality`: Pick the highest quality option across all tiers, regardless of cost (within budget).
- `balanced`: Score all options: `0.4 * quality + 0.3 * (1 - normalized_cost) + 0.3 * (1 - normalized_latency)`. Pick highest score.

## Budget

```
SESSION_BUDGET: $10.00
MAX_PER_CALL: $0.50
REMAINING: $10.00
STRATEGY: balanced
```

Only `pay` and `delegate` decisions deduct from the budget. If REMAINING < cost of a `pay` option, that option is unavailable — fall back to a lower tier.

## Cache

Maintain a cache of results from this session. Before any decision, check the cache:

```
| Capability | Params (key) | Result | Tier | At |
|------------|-------------|--------|------|----|
| web-scrape | url:example.com | {title: "...", ...} | use | 14:02 |
| email-lookup | person:Jane,company:Acme | jane@acme.com | pay | 14:05 |
```

Cache hit rules:
- Exact same capability + same params → return cached result, tier = `cache`
- Same capability + very similar params → mention the cached result, ask if it's sufficient before re-executing
- Cache is session-scoped — it resets when the conversation ends

---

## Capabilities

### email-lookup

Find a professional email address for a person at a company.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Web search | Search `"{person name}" "{company}" email` using web search tool | 0.50 | ~5s |
| Company website | Fetch the company's contact/team page | 0.40 | ~3s |

**gen option:**
- Quality: 0.60 | Latency: ~5s
- Instructions: Resolve the company's domain. Try common email patterns (`first@domain`, `first.last@domain`, `flast@domain`). Use web search to verify if possible.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Clearbit | `https://person-stream.clearbit.com/v2/combined/find` | $0.05 | 0.95 | 200ms |
| Hunter | `https://api.hunter.io/v2/email-finder` | $0.03 | 0.85 | 150ms |

**delegate option:**
- When: All automated options have failed or returned low-confidence results
- How: Ask the user to manually verify or provide the email address

---

### pdf-extract

Extract structured data from a PDF document.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Built-in PDF reader | Use your PDF reading tool to read the document directly | 0.75 | ~2s |
| `pdftotext` CLI | Run `pdftotext input.pdf -` locally | 0.70 | ~1s |

**gen option:**
- Quality: 0.80 | Latency: ~8s
- Instructions: Read the PDF with your built-in tools. Identify document type (invoice, contract, form). Extract key fields into structured JSON. Parse tables by aligning columns.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| DocumentAI | `https://documentai.example.com/v1/extract` | $0.10 | 0.98 | 3s |

**delegate option:**
- When: PDF is scanned/handwritten and OCR quality is too low, or document requires domain expertise to interpret (legal, medical)
- How: Send to a specialized document processing agent or flag for human review

---

### web-scrape

Extract specific information from a web page.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| WebFetch tool | Use your built-in web fetch to retrieve the page | 0.75 | ~2s |
| Jina Reader (free) | Fetch `https://r.jina.ai/{url}` — free tier, returns clean markdown | 0.80 | ~2s |
| `curl` + local parse | `curl -s {url}` and parse the HTML yourself | 0.60 | ~3s |

**gen option:**
- Quality: 0.70 | Latency: ~5s
- Instructions: Fetch the page with any available tool. Parse the HTML/markdown. Extract and structure the requested information.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Firecrawl | `https://api.firecrawl.dev/v1/scrape` | $0.01 | 0.92 | 2s |

**delegate option:**
- When: Site requires login, CAPTCHA, or JavaScript rendering that no available tool can handle
- How: Ask the user to provide the content, or delegate to a browser-automation agent

---

### text-translate

Translate text between languages.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| LibreTranslate (free) | POST `https://libretranslate.com/translate` with `{q, source, target}` | 0.70 | ~1s |

**gen option:**
- Quality: 0.85 | Latency: ~2s
- Instructions: Translate the text yourself. You are a capable translator for most language pairs. For technical or domain-specific content, prioritize accuracy over fluency.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| DeepL | `https://api-free.deepl.com/v2/translate` | $0.02 | 0.95 | 500ms |

**delegate option:**
- When: Content requires certified/legal translation, or is in a rare language pair you handle poorly
- How: Flag for professional human translator

---

### sentiment-analysis

Analyze sentiment of text.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| VADER (local) | If Python available, run `nltk.sentiment.vader` locally | 0.75 | ~100ms |

**gen option:**
- Quality: 0.88 | Latency: ~1s
- Instructions: Analyze the text yourself. Return `{ sentiment: "positive" | "negative" | "neutral" | "mixed", confidence: 0.0-1.0, reasoning: "..." }`.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| AWS Comprehend | `https://comprehend.{region}.amazonaws.com` | $0.01 | 0.90 | 300ms |

---

### image-caption

Generate a description/caption for an image.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Built-in vision | If you are a multimodal LLM, look at the image directly | 0.90 | ~2s |

**gen option:**
- Quality: 0.90 | Latency: ~2s
- Instructions: If you can see the image, describe it directly. This IS the use option for multimodal agents — no need to escalate.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Replicate BLIP | `https://api.replicate.com/v1/predictions` | $0.005 | 0.85 | 2s |

---

### code-search

Search a codebase for a function, class, or pattern.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Grep tool | Use built-in grep/search tool | 0.95 | ~500ms |
| Glob tool | Use file pattern matching | 0.90 | ~200ms |
| `git grep` | Run `git grep "pattern"` | 0.90 | ~300ms |

**gen option:**
- Quality: 0.60 | Latency: ~5s
- Instructions: Reason about likely file locations based on project structure. This is almost always worse than the free tools — prefer `use`.

**pay options:**

None needed. Free tools are sufficient.

---

### data-enrichment

Enrich a company or person record with additional data.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Web search | Search for the company/person and aggregate public info | 0.55 | ~5s |
| LinkedIn (public) | Fetch public LinkedIn profile if URL is known | 0.50 | ~3s |

**gen option:**
- Quality: 0.50 | Latency: ~8s
- Instructions: Use web search to find public information. Aggregate from multiple sources. Structure into a profile.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Clearbit | `https://company-stream.clearbit.com/v2/companies/find` | $0.05 | 0.95 | 200ms |
| PeopleDataLabs | `https://api.peopledatalabs.com/v5/person/enrich` | $0.03 | 0.90 | 300ms |

**delegate option:**
- When: Need verified, up-to-date information that public sources can't provide
- How: Delegate to a research agent with database access, or flag for manual research

---

### complex-analysis

Perform deep analysis requiring multiple data sources, domain expertise, or extended reasoning.

**use options:**

None typically available for complex analysis.

**gen option:**
- Quality: 0.70 | Latency: ~30s
- Instructions: Break the problem into sub-tasks. Use available tools for data gathering. Synthesize and analyze. This works for general analysis but may lack domain depth.

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Perplexity | `https://api.perplexity.ai/chat/completions` | $0.05 | 0.85 | 5s |

**delegate option:**
- When: Analysis requires specialized domain expertise (legal, financial, scientific), access to proprietary data, or human judgment
- How: Spawn a specialized agent with relevant tools/knowledge, or escalate to a human expert with a clear brief of what's needed and what you've gathered so far

---

## x402 Payment Flow

When you choose a `pay` option:

1. Make an HTTP request to the provider's endpoint
2. If the response is `402 Payment Required`, read the payment details from the response
3. Sign the payment with the agent wallet
4. Retry the request with the `X-PAYMENT` header containing the signed payment
5. Receive the result and deduct cost from budget

If x402 is not available, fall back to standard API auth if credentials exist in environment, otherwise fall back to `gen` or `use`.

## Delegation Protocol

When you choose `delegate`:

1. Clearly describe what needs to be done and why you can't do it at a lower tier
2. Include all context gathered so far (don't make the delegate redo your work)
3. Specify the expected output format
4. If delegating to a human: present it as a clear request, not a raw data dump
5. If delegating to another agent: use the Agent tool or relevant orchestration mechanism
6. Log the delegation and its cost (if any) in the spend log

## Spend Log

After every decision, record:

```
| # | Capability | Tier | Source | Cost | Remaining |
|---|------------|------|--------|------|-----------|
| 1 | web-scrape | cache | (previous #3) | $0 | $10.00 |
| 2 | email-lookup | use | web search | $0 | $10.00 |
| 3 | web-scrape | use | Jina Reader | $0 | $10.00 |
| 4 | data-enrichment | pay | Clearbit | $0.05 | $9.95 |
| 5 | complex-analysis | delegate | research agent | $0.20 | $9.75 |
```

Maintain this log in your working memory. Report it when the user asks about spend or decisions.

---

## Adding New Capabilities

Append a new section under `## Capabilities`:

```markdown
### capability-name

Description of what this does.

**use options:**

| Source | How | Quality | Latency |
|--------|-----|---------|---------|
| Free tool | How to use it | 0.XX | ~Xs |

**gen option:**
- Quality: 0.XX | Latency: ~Xs
- Instructions: How the agent should do it itself...

**pay options:**

| Provider | Endpoint | Cost | Quality | Latency |
|----------|----------|------|---------|---------|
| Name | `https://...` | $X.XX | 0.XX | Xms |

**delegate option:**
- When: Under what conditions to escalate
- How: How to hand it off
```

Any tier can be omitted if no option exists for it.
