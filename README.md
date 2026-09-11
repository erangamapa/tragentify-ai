# Tragentify

**The agentic trading that was promised.**

Tragentify is not a call group blasting tickers or a wallet tracker that copies strangers into rugs. It's your private team of agents for degen trading — new launches, memecoins, whatever's moving. They run on your rules.

---

## The noise you're trading against

Forty Telegram groups. Two hundred accounts on X. Discord servers you pay for. Launches every minute, most of them rugs. Threads where the useful part is the fourteenth reply. Screenshots instead of contract addresses. Alpha that's only alpha for ninety seconds.

**Keeping up with it is a full-time job, and people still lose.** They miss the one that mattered because they were asleep, or in a meeting, or three groups over. They buy the loud one because that's the one they saw. They size wrong, hold too long, and never write down why they bought it.

## What Tragentify is

**Your private team of agents, making the best trades on your rules.**

Instead of you wading through the noise across call groups and crypto Twitter.

They watch your sources and the chain continuously — every tweet, every group message, every launch, screenshots included. Ask them anything, or let them bring things to you. They have the whole picture; you set the rules they work to.

And they stay quiet the rest of the time, which is most days.

## How it behaves

- **Ask them anything.** Paste a contract and get a verdict in seconds. Ask for tokens matching whatever criteria are in your head. Ask what they looked at today and passed on.
- **It knows your rules.** You write a mandate once — how aggressive, what's off-limits, how much you'll risk. Every call cites it, and it refuses trades that break it even when you're the one asking.
- **It calls the exit as hard as the entry.** Every position carries a thesis and an invalidation, watched continuously. Liquidity moves at 3am and you hear about it.
- **You choose how much to hand over.** Tell me and I'll trade it myself · act but ask first · act and report back. Changeable any time, revocable in one message.
- **It tells you which of your sources actually make money.** The forty groups you follow get scored on outcomes. Most of them are noise, and it will say so.

## Status

**Planning complete. Nothing built yet.**

| | |
|---|---|
| Chain | Robinhood Chain · mainnet `4663` · testnet `46630` |
| Language | TypeScript, Node 22 |
| Scope | MVP is advisory — the desk calls, you trade. Execution in P4 |
| Build cost | ~$175/month through the go/no-go milestone |

## Documents

Read them in this order.

| Doc | What it is |
|---|---|
| [`docs/01-requirements.md`](docs/01-requirements.md) | What the product must do. 26 sections, ~260 numbered requirements with stable IDs (`DEF-1`, `SRC-12`, `EXIT-7` …). The contract. |
| [`docs/02-implementation-plan.md`](docs/02-implementation-plan.md) | How to build it. Named stack, named vendors, tiers and costs, database schema, frozen interfaces, 21 work packages sized for parallel handoff. |
| [`docs/03-brand.md`](docs/03-brand.md) | Name, tagline, description, positioning, voice rules. |

Requirement IDs are referenced throughout the implementation plan. When a work package says `Meets: SRC-20 … SRC-26`, those are clauses in doc 01.

## Non-negotiables

These are load-bearing. Breaking any of them breaks the product or the business.

- **`SCA-1` — shared pipeline, personal decision.** Ingestion and analysis happen once; only sizing and phrasing are per user. Run agents per user instead and inference cost kills the product at roughly fifty users.
- **`COST-2` — no model in the hot path before t+35s.** Reading the volume is arithmetic. The model only meets what already survived.
- **`REL-1` — exits survive a total LLM outage.** Exit monitoring, stops and kill switches are deterministic code with no model dependency. Someone is holding a position; the desk going quiet is not acceptable.
- **`DESK-1` — the risk engine is code, not a model.** Anything that can be talked out of a limit is not a risk manager.
- **`MEM-1` / `SEC-7` — private-source isolation.** Signal from a user's private group never reaches another user. Enforced in the data model, with a build-failing test.
- **`DEF-3` — the desk must be able to say nothing.** "Nothing worth your attention today" is the product working. No one whose revenue depends on activity can send that message.

## Build order

`P0` foundations → `P1` see → **`P2` select — the go/no-go** → `P3` speak → `P4` execute → `P5` harden

**P2 answers the only question that matters** — are the calls any good — with no signing service, no order engine and no custody. Execution is deliberately deferred: Maestro and Banana Gun already support this chain, so the first track record can be produced by executing calls by hand while the desk proves itself.

Do not staff P4 until P2 has produced a track record worth publishing.

## Verify first

Two one-minute checks that would invalidate real work if wrong.

1. **Permit2 deployed on 4663** — `eth_getCode` against the canonical address. The entire subscription mechanism depends on it.
2. **Alchemy's free tier survives a persistent WebSocket** — if not, that's $49/month from week one rather than during P3.

---

*Private repository. See `CLAUDE.md` for how to pick this up in a new session.*
