# Plan

## Rename

Rename the project from `use-gen-pay` to **openprocurement**.

- **Open** — community-driven, shared decisions, collective intelligence
- **Procurement** — the cascade, getting what you need at the right cost

- [x] Rename repo, skill file, references, and README

## What we have

A procurement cascade for AI agents:

```
CACHE → USE → GEN → PAY → DELEGATE
```

Single skill file, no dependencies, framework-agnostic.

## What we're adding

### 1. Community Intelligence Layer

A shared platform that aggregates cascade decisions from all users. Not a new tier — a layer underneath that makes every existing tier smarter.

- USE: community ranks which free tools actually work best
- GEN: community signals whether generating is good enough or don't bother
- PAY: community surfaces which paid providers give best value for cost
- DELEGATE: community shows when delegation was necessary and to whom

### 2. The Platform (GitHub-native)

GitHub repo is the platform. No separate backend needed.

- People submit decision records as **PRs**
- Community reviews and discusses in PR comments
- Merged records become the shared knowledge base
- Agents query the repo directly — it's just files

Each record looks like:

```
capability: pdf-extract
chosen_tier: PAY
provider: google-documentai
cost: $0.03
quality: 0.95
context: 12-page contract with tables
alternative_tried: GEN (quality: 0.60)
```

## Work

1. [x] Rename project to openprocurement (repo, skill file, references, README)
2. [x] Define the decision record schema (`decisions/_schema.yml`)
3. [x] Set up repo structure for community-submitted decision records (`decisions/`)
4. [x] Update skill file — integrate community intelligence into each tier's decision logic
5. [x] Make it so agents auto-post their decisions as PRs after each cascade run
