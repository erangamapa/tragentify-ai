# Tragentify

**The agentic trading that was promised.**

Tragentify is not a call group blasting tickers at five thousand strangers, and not a wallet tracker that copies someone you've never met straight into getting rugged. It's your own team of expert agents for degen trading on Robinhood Chain — seconds-old launches, memecoins, whatever's moving — in a private channel that's yours alone. They run on your rules, not ours: you set the mandate, and they refuse trades that break it, including when you're the one asking. Advise or execute, your call, revocable in one message. Most days they'll tell you there's nothing worth buying.

---

## Status

**Planning complete. Nothing built yet.**

| | |
|---|---|
| Chain | Robinhood Chain · mainnet `4663` · testnet `46630` |
| Language | TypeScript, Node 22 |
| Scope | MVP is advisory — Book C origination, no execution until P4 |
| Build cost | ~$175/month through the go/no-go milestone |

## Documents

Read them in this order.

| Doc | What it is |
|---|---|
| [`docs/01-requirements.md`](docs/01-requirements.md) | What the product must do. 26 sections, ~260 numbered requirements with stable IDs (`DEF-1`, `SRC-12`, `EXIT-7` …). The contract. |
| [`docs/02-implementation-plan.md`](docs/02-implementation-plan.md) | How to build it. Named stack, named vendors, tiers and costs, database schema, frozen interfaces, 21 work packages sized for parallel handoff. |
| [`docs/03-brand.md`](docs/03-brand.md) | Name, tagline, description, positioning, voice rules. |

Requirement IDs are referenced throughout the implementation plan. When a work package says `Meets: SRC-20 … SRC-26`, those are clauses in doc 01.

## The one idea everything rests on

Every token spreads along a propagation curve: alpha groups → mid-tier groups → crypto Twitter → the crowd. Where you enter on that curve determines your return.

Call groups alert when a token is **loud**, because loud is trivially easy to detect — which means by the time the message reaches you, you are the exit liquidity.

**Tragentify classifies where a token sits on that curve and only speaks while it's early. Tokens already trending are actively suppressed.**

## Non-negotiables

These are load-bearing. Breaking any of them breaks the product or the business.

- **`SCA-1` — shared pipeline, personal decision.** Analysis is computed once per candidate and fanned out. Run agents per user instead and inference cost kills the product at roughly fifty users.
- **`COST-2` — no model in the hot path before t+35s.** ~95% of decisions are arithmetic over cached values. The LLM only meets a token that survived a 5,000,000-to-3 filter.
- **`REL-1` — exits survive a total LLM outage.** Exit monitoring, stops and kill switches are deterministic code with no model dependency.
- **`DESK-1` — the risk engine is code, not a model.** Anything that can be talked out of a limit is not a risk manager.
- **`MEM-1` / `SEC-7` — private-source isolation.** Signal derived from a user's private group never reaches another user. Enforced in the data model, with a build-failing test.
- **`DEF-3` — the desk must be able to say nothing.** "Nothing cleared the bar today" is the product working. A call group cannot send that message; its revenue depends on activity.

## Build order

`P0` foundations → `P1` see → **`P2` select — the go/no-go** → `P3` speak → `P4` execute → `P5` harden

**P2 answers the only question that matters** — are the calls any good — with no signing service, no order engine and no custody. Execution is deliberately deferred: Maestro and Banana Gun already support this chain, so the first track record can be produced by executing calls manually while the classifier proves itself.

Do not staff P4 until P2 has produced a track record worth publishing.

## Verify first

Two things to check before writing code. Both are one-minute checks that would invalidate real work if wrong.

1. **Permit2 deployed on 4663** — `eth_getCode` against the canonical address. The entire subscription mechanism depends on it.
2. **Alchemy's free tier survives a persistent WebSocket** — if not, that's $49/month from week one rather than during P3.

---

*Private repository. See `CLAUDE.md` for how to pick this up in a new session.*
