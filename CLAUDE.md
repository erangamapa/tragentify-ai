# Tragentify — context for a new session

Read this first, then `docs/01-requirements.md` and `docs/02-implementation-plan.md`.

## What this is

**Degen trading is an information problem before it is a trading problem.** Forty Telegram groups, hundreds of accounts on X, paid Discords, launches every minute, most of it noise. Nobody can process that fast enough to act well.

Tragentify is **a private team of agents per user that reads all of it and returns decisions instead of information** — what is worth buying, why now, how much against the rules that user set, and when to get out. Delivered in a private Telegram channel per user, on **Robinhood Chain** (`4663`). MVP is **advisory**: the desk researches, argues and calls; the user trades. Execution comes in P4.

**Frame it as digestion, not detection.** Finding tokens is trivial — the chain lists every one. The product is deciding which of thousands deserve a person's attention and saying so fast enough to matter. If copy or a feature drifts toward "we filter better than the other guys," it has lost the plot.

Nothing is built. The two docs are the complete specification and they were argued through in detail — treat them as decided, not as suggestions.

## How the docs work together

`docs/01-requirements.md` holds ~260 numbered requirements with stable IDs. `docs/02-implementation-plan.md` holds 21 work packages, each naming the requirement IDs it satisfies. When implementing a package, read its requirement clauses first — the *why* is in doc 01, the *how* in doc 02.

## Rules that are not negotiable

Several decisions look like preferences and are not. Changing any of these breaks the cost model, the latency budget, or the product's reason to exist.

- **No model runs in the hot path before t+35s.** Kill filters, dedupe, velocity, propagation classification, scoring, risk checks, sizing, exits, execution and payments are all plain code. If a tier-0 task acquires a model call, the economics stop working.
- **Analysis is computed once per candidate, then fanned out per user.** Never per-user agent runs.
- **`packages/risk` and `apps/exit-monitor` must not import `packages/model`.** Enforced by an ESLint boundary rule. Exits have to survive a total LLM outage.
- **No agent framework.** No ElizaOS, no LangChain, no LangGraph, no CrewAI. `ModelClient` + zod + BullMQ, about 200 lines. The Vercel AI SDK is used as a primitive inside `ModelClient`, never as an orchestrator. Reasons are in doc 02 §2.
- **Model tiers are configuration in `packages/model`, never hard-coded names.** The tier map is doc 02 §7.
- **Private-source signal never crosses users.** Row-level security in Postgres, with a test that fails the build if it leaks.
- **Raw message text is not memory.** Parse to structured facts, discard the text, keep a reference for re-fetching context on survivors.

## Where to start

`WP-01` (foundations) blocks everything. It publishes `packages/contracts` — the frozen interfaces in doc 02 §9 — and those must be merged before any other package is assigned. After that, eight packages run fully in parallel.

Build the **replay harness** during WP-01, not later. Re-running historical launches through any pipeline stage is what makes the classifier tractable. Robinhood Chain mainnet only opened 1 July 2026, so the full history is short enough to replay end to end — an advantage that shrinks as the chain ages.

## Delegating to a team

Doc 02 §12 has the waves. In short: freeze the contracts, hand one package per assignee with its requirement IDs and its done-when condition written as an executable test, and keep three packages in-house — **WP-08** (propagation classifier), **WP-12** (desk orchestration), **WP-17** (exit triggers). One is the product; the other two are the ways to lose money.

## Verify before building

- **Permit2 deployed on 4663** — the whole subscription mechanism rests on it
- **Alchemy free tier vs a persistent WebSocket** — decides whether that line is $0 or $49 from week one

## Conventions

- TypeScript strict mode, functional patterns, single quotes, semicolons
- Commits: `[Type][TSK-0000] Description`
- Never `--no-verify` on commit
- Money is `numeric` in Postgres, never float; addresses stored checksummed
- `data-test-id` for test selectors, `import` never `require`
