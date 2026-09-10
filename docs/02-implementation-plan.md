# Tragentify Build Plan

*Tragentify · tragentify.ai · Implementation Plan v0.6 · Robinhood Chain*

Named stack, named vendors, keys, schema and twenty-one work packages with frozen interfaces — sized so independent engineers or agents can build them in parallel without coordinating on anything but the contracts in §9.

**Status** Ready to assign · **Date** 11 Sep 2026 · **Chain** Robinhood Chain · 4663 · **Language** TypeScript

---

## 01. Chain facts that shape the build

Robinhood Chain is not a generic EVM L2, and four of its properties change the architecture directly. Read this section before anything else — several decisions downstream only make sense in light of it.

| Fact | Consequence for us |
|---|---|
| Single Robinhood-run sequencer, **first-come-first-served** ordering, no priority-fee auction, no public mempool | No MEV bidding layer to build. No Jito or Flashbots equivalent. **Latency to the sequencer is the only lever**, which makes co-location the whole execution strategy rather than fee bidding |
| A **sequencer feed over WebSocket** (Nitro relay) exposes transactions before they execute | This is our primary ingest primitive and it is strictly better than a mempool. We see buys queued before they land. Chainstack open-sourced a decoder — start from it |
| A **compliance engine runs inside the sequencer**, simulating each transaction before sequencing | Transactions can be rejected at chain level for reasons unrelated to our logic. Every failure path must distinguish compliance rejection from ordinary revert, and surface it honestly to the user |
| Uniswap v4 is the primary AMM and **hooks are the dominant honeypot vector** | Hook bytecode analysis replaces the LP-lock checks a Solana build would use. A malicious hook can permit buys and block sells while the pool looks clean |
| 1,000+ tokenized equities and ETFs as plain ERC-20s, alongside permissionless launchpads | Books A and B trade a real RWA universe; Book C trades fresh mints. One chain, three books, which is exactly what the desk architecture is for |
| ETH gas, ~100ms blocks, chain ID **4663** mainnet / **46630** testnet, ERC-4337 supported | Testnet is a genuine dev environment. Account abstraction makes scoped session keys viable, so the non-custodial path is real rather than aspirational |
| Aggregators live from day one: **1inch, 0x, LI.FI, UniswapX** | We never write a router. Quote across two aggregators and take the better fill |

> **The single most important consequence.** On a first-come-first-served chain with no fee auction, being early is purely a function of how fast you detect and how close you sit to the sequencer. Every millisecond in the detect-to-submit path converts directly into fill quality — which is a far more tractable engineering problem than winning a gas auction, and it is where this build should spend its effort.

## 02. Stack decisions

Decisions, not options. Robinhood Chain only for v1 — no other chain, no multi-chain abstraction beyond keeping `ChainAdapter` honest.

| Layer | Choice | Why |
|---|---|---|
| Language | TypeScript 5.7, Node 22 LTS | Strict mode throughout. One language across bots, workers and web |
| Monorepo | pnpm workspaces + Turborepo | Shared types across 11 deployables without publishing packages |
| Chain client | viem 2.x | Typed, fast, tree-shakeable. Custom chain definition for 4663 |
| Chain ingest | Nitro sequencer feed over WebSocket | Pre-execution visibility. Fork `chainstacklabs/robinhood-chain-sequencer-feed` rather than writing a decoder |
| RPC — reads | Alchemy | Live on 4663 with WebSocket. Public `rpc.mainnet.chain.robinhood.com` is the free fallback — degraded, but it satisfies `REL-3` without a second paid provider |
| RPC — writes | Dedicated low-latency endpoint, region-pinned | Separate from read traffic so read load can never delay a submit |
| Swap routing | 1inch Swap API + 0x Swap API | Quote both, take the better. UniswapX and LI.FI Intents as v2 additions |
| Token data | Bitquery Robinhood Chain API | Stock tokens and DEX data on this chain specifically. Historical price and liquidity for backfill scoring |
| X — real time | TweetStream (WebSocket) | Push, not polling. Sub-second detection on tracked accounts, **OCR on image posts**, contract and ticker extraction already done. 1322 is the alternative if multi-platform coverage matters |
| X — backfill | twitterapi.io | Different job entirely: bulk history for source scoring, where latency is irrelevant and cost per tweet is everything |
| Narrative layer | In-house keyword velocity | The same Count-Min Sketch already counting contract mentions, extended to keywords and phrases weighted by source score. Zero marginal cost. Applied as a **scoring multiplier on matching candidates, never as a trigger** — themes move over days, Book C moves in minutes |
| Explorer / verification | Blockscout API | Source verification and contract metadata for forensics |
| Database | Neon Postgres 17 + pgvector | Branch per PR. pgvector avoids a separate vector store |
| Migrations | Drizzle ORM + drizzle-kit | TS-native schema, generated types |
| Hot cache | Redis 7, self-hosted, co-located | Sub-millisecond. Never an HTTP-based Redis in the hot path |
| Queues | BullMQ | Redis-backed, TS-native, DLQ and delayed jobs built in |
| Hosting — workers | Fly.io, single region, sequencer-pinned | Region chosen by measurement, not assumption — see WP-02 |
| Hosting — web | Vercel (Next.js 15) | Marketing, Mini App payment surface, admin |
| Telegram bots | grammY | One bot instance per agent identity |
| Telegram MTProto | GramJS | Concierge provisioning and user-session readers. Telethon is the Python fallback at pool scale |
| Discord — admin servers | discord.js | Bot-invite path, for the rare case the user runs the server |
| Chat OCR | Tesseract, self-hosted | Screenshot calls are as common in Telegram and Discord as on X, and no vendor covers them. Runs on the Fly workers, free. Reading a contract address off an image is well within what it does reliably |
| Discord — member servers | discord.js-selfbot-v13 | Gateway client on the user's own account token — the only way to reach an invite-only group. Read-only, passive, channel-allowlisted. ToS risk sits with the user's account and is disclosed before we ask |
| Key management | Turnkey | Keys in AWS Nitro Enclaves, policy engine, EVM-native. **Needed from P3** — the subscription charger must sign `transferFrom`. Pay-as-you-go until trading keys arrive in P4 |
| Smart accounts | ERC-4337 via Alchemy Account Kit | Session keys with contract and asset allowlists — the non-custodial default from `WAL-2` |
| Agent framework | None — deliberate | `ModelClient` + zod + BullMQ, roughly 200 lines. Reasons below |
| Structured output | Vercel AI SDK (`ai`) | `generateObject` with zod schemas, used as a primitive *inside* `ModelClient` — never as an orchestrator |
| Model gateway | OpenRouter → LiteLLM | Start managed; self-host when monthly spend passes roughly $2k |
| Agent memory | Mem0 (mem0ai) on pgvector | Swap to Zep only when temporal reasoning becomes the bottleneck |
| Payments | Permit2 allowance on a stablecoin | User signs, we pull. Canonical singleton — no contract to deploy. No cards, no merchant of record, no tax registration |
| Wallet connect | wagmi + viem + RainbowKit | One page. Auto-switches to 4663 and adds the chain if unknown |
| Payment surface | Telegram Mini App | Keeps the flow inside Telegram — the in-app browser breaks WalletConnect returns. Plain web page is the fallback, not the target |
| Cost attribution | Postgres `model_calls` table | One row per call — user, task, tier, tokens, cost. Roughly 50 lines, satisfies `COST-7`. Langfuse only when spend justifies the tooling |
| Errors / logs | `pino` → Axiom, free tier | A pino transport, roughly ten lines. 500 GB/month free against maybe 5 GB of use — free well past 1,000 users. Gives searchable history, so an overnight failure is not lost the way it is with `fly logs`. **Not Sentry** — the failures here are silences, not exceptions, and Sentry is blind to silence by design |
| Secrets | Fly secrets + Vercel env | Enough until there is a team. Infisical when more than a couple of people need access |
| CI/CD | GitHub Actions | Turborepo remote cache, Neon branch per PR |
| Testing | Vitest + Testcontainers + Anvil fork | Fork 4663 locally for execution tests without spending real gas |

> **Why no agent framework.** This is not an agentic loop. Each agent reads specific precomputed data, makes one model call against a tight prompt, and returns output validated by zod. No planning, no autonomous tool selection, no model deciding what runs next — **BullMQ is the orchestrator** and the desk conversation is a rendering format over a deterministic pipeline. Telegram reinforces this: bots cannot read each other's messages, so the "debate" was always backend-composed.

**Specifically rejected.** `ElizaOS` — character and plugin oriented, built for social agents, fights every determinism requirement here. `LangGraph` — Python-first, weaker JS port, heavy for a DAG we already own. `CrewAI` and `AutoGen` — role-playing multi-agent conversation is precisely what we are not doing. A framework would blur per-task cost attribution (`COST-7`), fight per-tier routing (`COST-1`), and add latency inside a 35-second budget.

**Where to revisit.** Ask-the-desk is the one path that could become a real agent loop — query the chain, then decide to check social, then decide to simulate. If it outgrows three fixed steps, use the AI SDK's tool-calling on that path alone, with a hard step cap. Never in the hot path, never near execution.

## 03. Third-party services

> **The build runs at roughly $175/month.** That covers P0 through early P2 — including **M2, the go/no-go on whether source scoring works at all**. The expensive subscriptions only begin once the desk needs live detection, which is after the question that matters has already been answered.

### Stage 1 — Build · P0 to early P2 · ~$175/mo

Everything here is either free, free-tier, or the cheapest paid tier. Source scoring at this stage is **entirely backfill** — historical tweets and historical prices — so nothing needs to be live yet.

| Service | Tier to buy | Cost | Why this tier |
|---|---|---|---|
| TweetStream | 3-day trial | $5 | 5 accounts, 1 WebSocket, stablecoin deposit credited on conversion. Enough to validate the payload contract, confirm OCR actually reads screenshot calls, and measure real latency on 4663 addresses. **Do not subscribe yet** |
| twitterapi.io | Pay as you go | ~$40 | Backfill-heavy at this stage — pulling months of history for 200+ candidate sources at ~$0.15/1k tweets. This is the workhorse of M2 |
| Alchemy | Free → Growth | $0–49 | Start free. Move to Growth the moment a persistent WebSocket subscription exceeds the free compute allowance — verify this early, it may bite in week one |
| Fly.io | Pay as you go | ~$30 | Two or three shared-CPU machines plus a small Redis. Full 11-worker deployment comes later |
| Neon | Launch | $19 | Branching for per-PR test databases is the feature you need; Scale is unnecessary at this volume |
| GeckoTerminal *or* Bitquery | Free → Pro | $0–49 | Try GeckoTerminal free first — if its history depth covers scoring, the Bitquery line stays at zero |
| OpenRouter | Credits | ~$30 | Mostly T1 batch work: bot classification during source scoring. T3 barely runs before the desk exists |
| Vercel | Hobby | $0 | Nothing commercial is shipping yet |
| Axiom | Personal | $0 | Free forever, no card. 500 GB/month ingest against ~5 GB of real use. Timebox the integration to half an hour and build no dashboards |
| GoPlus · Chainalysis · Blockscout · DexScreener · 1inch · 0x · Permit2 · Telegram · Discord · GitHub Actions · public RHC RPC | Free | $0 | All free, all load-bearing |
| **Total** |  | **~$175** |  |

> **Concierge accounts — needed at WP-14, not before.** A Telegram bot cannot create a group, so provisioning is driven by real Telegram user accounts over MTProto. Each needs a phone number to register — **a one-time setup for the machine, not a per-user cost**. Two or three accounts cover the first tens of users; the pool grows only when signup volume demands it. Register them when WP-14 starts. The one caveat worth knowing now: aged accounts survive Telegram's spam detection better than accounts registered that morning, so register them a little ahead of needing them rather than on launch day.

### Stage 2 — Live · late P2 to P3 · ~$400/mo

Triggered by one event: the desk starts calling trades in real time for the manual track record. Until that moment, none of these upgrades buy anything.

| Change | Tier | Cost | Trigger |
|---|---|---|---|
| TweetStream | Minimum | $199 | Live detection begins. 50 accounts, 3 connections. **Not Pro** — see the routing rule below |
| twitterapi.io | Pay as you go | ~$20 | Drops as backfill finishes and it becomes the long-tail polling lane |
| Fly.io | Scale up | ~$50 | Full worker set deployed |
| OpenRouter | Credits | ~$50 | The desk exists, so T2 and T3 start running |
| Vercel | Pro | $20 | Mini App ships commercially |
| Turnkey | Pay as you go | $0 → $10 | **P3.** Charger key signs the subscription pull. Free tier covers the first 25 users |

> **The routing rule that keeps TweetStream at $199.** Source score decides which lane a handle sits in — the top 50 get sub-second push with OCR, everything else gets polled at 20–30 seconds through twitterapi.io for a few dollars. Handles are promoted into the fast lane when they earn it and demoted when they stop. User-added handles land in the polling lane by default, which is correct anyway since they arrive unscored. Move to Pro at $499 only when more than 50 handles simultaneously justify sub-second delivery — a good problem, and a rounding error against revenue by then.

### Stage 3 — ~100 users · ~$1,246/mo

| Service | Tier | Cost |
|---|---|---|
| OpenRouter | Credits | ~$300 |
| Alchemy | Scale | $289 |
| TweetStream | Minimum | $199 |
| Fly.io | Pay as you go | ~$200 |
| Bitquery | Pro | $99 |
| Neon | Scale | $69 |
| twitterapi.io | Pay as you go | ~$60 |
| Vercel | Pro | $20 |
| Turnkey | Pay as you go | ~$10 |
| **Total** · ~$12.50 per user |  | **~$1,246** |

### Deferred to P5 — and what triggers each

| Service | Add when |
|---|---|
| AWS KMS *~$1/mo* | Before anyone beyond the founder has Fly access. Cheapest fix in the plan |
| QuickNode | An Alchemy outage costs real money |
| Langfuse | Model spend justifies the tracing UI. Until then `model_calls` satisfies `COST-7` |
| Infisical | More than a couple of people need secret access |
| Apify | twitterapi.io proves insufficient for backfill |
| TweetStream Pro *$499* | More than 50 handles simultaneously justify sub-second delivery |
| LunarCrush Individual *$90/mo* | In-house keyword velocity proves narrative detection is worth paying to widen. Its Topics endpoint buys breadth our source set cannot see. **Kaito does the same job at $833/mo and is not worth it** |

## 04. Secrets inventory

```
# Chain
RHC_CHAIN_ID=4663                # 46630 on testnet
RHC_SEQUENCER_FEED_WS            # Nitro relay endpoint
ALCHEMY_API_KEY
ALCHEMY_RHC_HTTP_URL
ALCHEMY_RHC_WS_URL
ALCHEMY_GAS_MANAGER_POLICY_ID    # ERC-4337 sponsorship
RHC_PUBLIC_RPC_URL               # free degraded failover
RHC_SUBMIT_RPC_URL               # dedicated write endpoint, region-pinned
BLOCKSCOUT_API_URL

# Market data and routing
BITQUERY_API_KEY
ONEINCH_API_KEY
ZEROX_API_KEY

# Social ingest
TWEETSTREAM_API_KEY            # real-time X push, OCR, token detection
TWITTERAPI_IO_KEY              # history backfill only
TELEGRAM_API_ID
TELEGRAM_API_HASH
TELEGRAM_CONCIERGE_SESSIONS      # JSON array of encrypted session strings
DISCORD_BOT_TOKEN              # admin-invited servers only
DISCORD_USER_TOKENS            # per-user account tokens, KMS-encrypted, same
                               # isolated store as TELEGRAM_CONCIERGE_SESSIONS

# Agent bot identities — one per desk member
TG_BOT_CHAIR_TOKEN
TG_BOT_FLOW_TOKEN
TG_BOT_FORENSICS_TOKEN
TG_BOT_NARRATIVE_TOKEN
TG_BOT_RISK_TOKEN
TG_BOT_MICROSTRUCTURE_TOKEN
TG_BOT_MACRO_TOKEN
TG_BOT_FUNDAMENTALS_TOKEN
TG_BOT_EXECUTION_TOKEN

# Models
OPENROUTER_API_KEY

# Wallets and signing — P4 only, not needed for advisory
TURNKEY_API_PUBLIC_KEY
TURNKEY_API_PRIVATE_KEY
TURNKEY_ORGANIZATION_ID
WALLET_ENCRYPTION_KEY_ID         # KMS key for imported-key envelope encryption

# Payments
PERMIT2_ADDRESS                  # canonical singleton on 4663 — verify deployment
SUBSCRIPTION_TOKEN_ADDRESS       # the stablecoin charged
SUBSCRIPTION_PRICE_UNITS         # monthly amount, token decimals
TREASURY_ADDRESS                 # destination of every pull
CHARGER_SIGNER_KEY_ID            # Turnkey key that submits transferFrom
SANCTIONS_API_KEY
WALLETCONNECT_PROJECT_ID

# Infrastructure
DATABASE_URL
REDIS_URL
AXIOM_TOKEN
AXIOM_DATASET
PROVISION_LINK_SIGNING_SECRET     # HMAC for one-time deep-link tokens
```

### Known on-chain addresses to configure

```
PONS_FACTORY   0x7ed598bcef8bd9edd8c97a195c6d13f40801ec7e   # emits TokenLaunched
PONS_ROUTER    0xe33e9e479df8802cb0866d5d05258bec4cf62948
UNIV4_POOLMANAGER  # Initialize events
UNIV3_FACTORY      # PoolCreated
UNIV2_FACTORY      # PairCreated
PLEIADES_FACTORY   # Robinhood-native AMM
POOLS_TRADE_FACTORY # Uniswap Labs launchpad, 1B fixed supply, locked liquidity
HOOD_FUN_FACTORY
Verify every address against Blockscout before use. Treat this list as
configuration in the database, not constants in code — launchpads churn.
```

## 05. Repository layout

```
tragentify/
├─ apps/
│  ├─ web/                 Next.js 15 — marketing, Mini App payment surface, admin
│  ├─ ingest-chain/        Sequencer feed decoder + log watchers on launchpad factories
│  ├─ ingest-social/       X poller, Telegram readers, Discord bot
│  ├─ filter/              Hot path — kill filters + enrichment. T0 only
│  ├─ classifier/          Clustering, propagation stage, candidate scoring
│  ├─ desk/                Agent orchestration, per-user fan-out, Chair
│  ├─ execution/           Order lifecycle, 1inch/0x quotes, Turnkey signing, submit
│  ├─ exit-monitor/        Deterministic exit engine. Separate deployable by design
│  ├─ tg-gateway/          Bot send/receive, per-group rate limiter, threads
│  ├─ provisioner/         MTProto concierge pool, group creation
│  └─ jobs/                Backfill scoring, attribution, reports, reconciliation
├─ packages/
│  ├─ db/                  Drizzle schema, migrations, query helpers
│  ├─ contracts/           Every shared type in §9. The frozen surface
│  ├─ model/               ModelClient — tiering, routing, caching, cost ledger
│  ├─ memory/              Mem0 wrapper, scope enforcement, retrieval budgets
│  ├─ chain/               ChainAdapter — Robinhood Chain impl over viem
│  ├─ forensics/           Token + Uniswap v4 hook bytecode analysis
│  ├─ wallet/              Turnkey wrapper, ERC-4337 session-key policies
│  ├─ reputation/          Deployer, wallet and source score reads/writes
│  ├─ risk/                Deterministic limit engine. No model calls, ever
│  └─ obs/                 pino → Axiom, correlation IDs, cost ledger
└─ infra/
   ├─ fly/                 One fly.toml per worker app
   └─ github/              Actions workflows
```

> **Two rules that keep the packages independent.** Nothing in `apps/` imports from another app — only from `packages/`. And `packages/risk` and `apps/exit-monitor` must have zero dependency on `packages/model`; an ESLint boundary rule enforces it, so a model outage can never reach the code that gets people out of positions.

## 06. Deployables

| Service | Responsibility | Scale by | Criticality |
|---|---|---|---|
| ingest-chain | Sequencer feed subscription, launchpad factory logs, Uniswap v4 `Initialize`, tracked wallet transfers | Singleton, hot standby | High |
| ingest-social | X polling, Telegram MTProto readers, Discord gateway | Shard by source group | Medium |
| filter | Kill filters, hook analysis, cached enrichment, confirmation window. No model calls | Horizontal, stateless | High |
| classifier | Cluster mentions to candidates, propagation stage, score, suppression | Horizontal, stateless | High |
| desk | Agent runs, Chair synthesis, per-user sizing and mandate check, composition | Horizontal, queue-driven | Medium |
| execution | Order validation, dual-aggregator quote, Turnkey sign, submit, confirm | Horizontal, partitioned by wallet | Critical |
| exit-monitor | Per-position triggers, ladders, decay, time stops. Fires sell orders | Horizontal, partitioned by position | Critical |
| tg-gateway | Outbound queue with per-group limits, inbound fan-in, thread mapping | Horizontal, partitioned by group | High |
| provisioner | Concierge pool, group creation, bot promotion, invite issue and revoke | Bounded by account pool | Medium |
| jobs | Source backfill, score recompute, attribution, reports, reconciliation | Cron, batch | Low |
| web | Marketing, wallet connect and Permit2 signing, deep-link binding, admin | Vercel | Medium |

### Event flow

```
sequencer feed ─┐
launchpad logs ─┼─> raw.events ─> filter ─> candidates.surviving ─> classifier
ingest-social ──┘         ^                                                │
                          │                                               v
                   forensics (hooks,                          candidates.scored
                   token bytecode)                                        │
                                             ┌────────────────────────────┤
                                             v                            v
                                     desk (fan-out per user)      suppression sink
                                             │
                         ┌───────────────────┼───────────────────┐
                         v                   v                   v
                   tg-gateway          orders.pending      memory writes
                   (ambient / tag)            │
                                              v
                                         execution ─> fills ─> exit-monitor
                                                                    │
                                                             orders.exit (priority)
```

### Signup and payment flow

```
Telegram   /start                     → we hold telegram_user_id. Free, before any payment.
             Chair: "One month, $99"
             [ Set up ]                 → single-use setup token
                │
Mini App   connect wallet             wagmi + WalletConnect
             auto-switch to 4663        wallet_switchEthereumChain, else addEthereumChain
             balance pre-check          zero stablecoin → swap link, do not dead-end
                │
             [ Pay for a month ]        allowance: one charge only, short expiry
             [ Pay monthly ]            allowance: recurring, ~6 months, wallet shows it
                │
             ONE signature              gasless. The signature also proves ownership —
                │                       no separate SIWE step.
                v
Backend    sanctions-screen payer     before access, not after
             permit2.transferFrom       we pay the gas — cents
             activate, bind wallet↔user
                │
Telegram   Chair: "You're in until 7 October."
             mandate conversation begins

Renewal: cron pulls monthly. Failed pull → plain notice → grace → deprovision.
Cancel = one message to the Chair, or revoke in the wallet. Both equal.
Dropped mid-flow? We still have their Telegram ID from step 1 — Chair follows up.
```

## 07. Ingestion and preprocessing

> **Nothing is summarised. Everything is extracted.** A summary is where detail goes to die, and detail is exactly what we cannot afford to lose. Instead, volume is destroyed by deterministic filters before any model exists in the path — 99% of it inside the reader process, at microsecond cost. Only the few candidates that survive ever reach an LLM, and they reach it as original text, not a digest.

### Seven layers

| Layer | Does | Runs | Cost |
|---|---|---|---|
| L0 · Vendor preprocessing | TweetStream: sub-second push, **OCR on image posts**, ticker and contract detection. Sequencer feed: decoded pre-execution transactions. Bought, not built | Vendor | Subscription |
| L1 · Extraction | Regex plus checksum validation, filtered to contracts that exist on 4663. Everything without an address dies here | Reader process | Microseconds |
| L2 · Dedupe | Bloom filter membership test before any database read | In-memory | O(1) |
| L3 · Aggregation | Count-Min Sketch for weighted velocity · HyperLogLog for distinct-source count · Redis sorted sets for sliding windows | Redis | Fixed memory |
| L4 · Kill filters | Hook verdict, token contract, deployer reputation, concentration, liquidity floor. Cache lookups only | filter worker | <100ms p99 |
| L5 · Classification | Propagation stage and composite score. Arithmetic over counters | classifier | Sub-ms |
| L6 · Desk | The first and only model call. Receives the **original text** of the surviving candidate's mentions, never a summary | desk | T2 / T3 |

### Attrition — why volume never becomes a cost problem

```
  ~5,000,000  messages, posts and transactions per day at 10k sources
        │        L1 — no contract address
        v
     ~40,000  carry an address
        │        L2 — already seen
        v
      ~2,000  distinct tokens
        │        L4 — hook, contract, deployer, liquidity
        v         ~95% killed
        ~100  survive to scoring
        │        L5 — propagation stage, suppression
        v
       3–15  reach the desk. These are the only ones a model ever sees.

Scale the top number by 100x and every layer below it is unchanged.
That is the property the whole cost model depends on.
```

### Two rules about text

- **PRE-1** — For **public sources** (X, base-set groups), a surviving candidate's originating message text is retained for the desk's narrative read, then dropped once the decision closes. The model sees what was actually written.
- **PRE-2** — For **private sources** (a user's own Telegram groups and Discord channels), only extracted signal ever leaves the reader. The desk receives the fact — *address X, source Y, time Z* — never the message. This is what makes `MEM-2` and the private-source isolation rule true rather than aspirational. `[HARD]`

### Where a model runs — and where it must not

Every task in the system is assigned a tier. Promotion to a higher tier requires evidence the lower one failed, and no package may invent its own model choice — tiers are configuration in `packages/model`, never code.

| Task | Tier | WP | Why |
|---|---|---|---|
| Kill filters, dedupe, velocity, scoring, risk checks, sizing, exit triggers, fills | T0 — none | 02–08, 11, 13, 16, 17, 19 | Arithmetic and cached lookups. A model here would be slower, dearer and less predictable |
| Bot and engagement-farm classification when scoring a source | T1 | 05 | Cheap pattern judgement, batch, latency irrelevant |
| Screening criteria → structured filter | T1 | 21 | Translation, not reasoning. Output validated by zod before it reaches Postgres |
| Source scores → readable report | T1 | 20 | Numbers into a sentence |
| Forensics verdict → one-line finding | T1 | 12 | The verdict is already computed; the model only phrases it |
| **Narrative read of actual post text** | T2 | 12 | **The one place raw text genuinely needs reading.** Why does this token exist, is anyone real behind it, is the attention organic |
| Ask-the-desk answers | T2 | 12 | Open-ended, user-facing, latency budget in seconds not milliseconds |
| Friday review and attribution prose | T2 batch | 18 | Scheduled, so batch pricing applies |
| Chair synthesis, sizing rationale, message composition | T3 | 12 | The decision itself. Worth the best model available — it runs a handful of times a day |
| Pre-mortem above the size threshold | T3 | 12 | Adversarial reasoning is exactly what a frontier model is for |
| Mandate conversation at onboarding | T3 | 18 | Once per user, and it sets every decision that follows. Never economise here |

> **Five places a model may never appear**, enforced by lint rule and review: anywhere in the hot path before t+35s · `packages/risk` · `apps/exit-monitor` · order construction and submission in `apps/execution` · provisioning and payment charging. The first is about latency and cost; the rest are about money moving on a decision that must be reproducible.

### What we buy versus build

**Bought:** L0 entirely — TweetStream for X streaming with OCR, and the forked Chainstack decoder for the sequencer feed. **Built:** L1 through L5, because they are 200 lines of deterministic code that no vendor can tailor to our kill filters or our propagation classifier. **Never built:** a summarisation layer. It would cost money, add latency, and lose the detail it exists to compress.

## 08. Data model

Core tables only. Drizzle schema in `packages/db`. Every table has `created_at`, `updated_at`. Money is `numeric`, never float. Addresses stored checksummed.

```
-- identity and commerce
users              id, email, telegram_user_id, tier, status, jurisdiction
subscriptions      id, user_id, provider, provider_ref, status, current_period_end
provision_tokens   token_hash, user_id, expires_at, consumed_at
channels           id, user_id, tg_chat_id, concierge_account_id, state

-- mandate and books
mandates           id, user_id, version, document jsonb, active
settings           id, user_id, book, suggested_size, max_slippage_bps,
                    gas_cap_wei, auto_execute_ceiling, order_ttl_ms,
                    max_open_positions, cooldown_ms

-- sources and scoring
sources            id, kind (telegram|discord|x), external_id, tier
                    (base|user_public|user_private), owner_user_id,
                    state (shadow|active|retired)
source_scores      source_id, window, calls, hits, median_return,
                    time_to_peak_s, rug_rate, updated_at
source_clusters    cluster_id, source_id, correlation
mentions           id, source_id, token_address, seen_at, raw_ref
                    -- raw_ref is a pointer, never the message text

-- reputation (mirrored in Redis)
deployers          address, launches, rugs, best_multiple, updated_at
wallets_tracked    address, pnl_30d, win_rate, tier, updated_at
hooks_known        hook_address, bytecode_hash, verdict
                    (safe|blocks_sell|taxes|unknown), analysed_at
                    -- Uniswap v4 hook registry. The chain-specific rug vector

-- assets
tokens             address, symbol, decimals, launchpad, pool_id,
                    hook_address, class (fresh|ecosystem|stock_token),
                    first_seen_block
                    -- class drives which book may hold it

-- candidates and calls
candidates         id, token_address, first_seen_at, propagation_stage,
                    score, kill_reason, suppressed_reason, state
                    -- retained for every candidate ever seen, including
                       killed ones. This table is what screening queries.
saved_screens      id, user_id, phrasing, filter jsonb, active, last_fired_at
                    -- an ad-hoc screen promoted to a standing watch
calls              id, candidate_id, user_id, book, thesis,
                    invalidation jsonb, proposed_size, exit_plan jsonb,
                    mandate_version, state
dissent            call_id, agent, position, rationale, scored_outcome

-- execution and positions
wallets            id, user_id, address, custody_mode
                    (generated|session_key|imported), turnkey_key_id,
                    smart_account_address
orders             id, call_id, user_id, wallet_id, side, token_address,
                    size_wei, max_slippage_bps, gas_cap_wei, ttl_expires_at,
                    idempotency_key UNIQUE, aggregator (1inch|0x),
                    state, failure_reason, failure_class
                    (revert|compliance_reject|ttl|slippage|nonce)
fills              id, order_id, tx_hash, price, qty, slippage_bps,
                    gas_paid_wei, venue, filled_at
positions          id, user_id, token_address, qty, cost_basis, thesis_id,
                    exit_plan jsonb, state, opened_at, closed_at
exit_triggers      position_id, kind (hard|decay|time|ladder),
                    params jsonb, armed, fired_at

-- memory
facts              id, scope (global|cohort|user), scope_ref, subject,
                    predicate, value jsonb, provenance jsonb, valid_from,
                    superseded_by, embedding vector(1536)
summaries          id, scope, scope_ref, period, body, token_count

-- cost, replacing Langfuse until spend justifies it
model_calls        id, user_id, task, tier, model, input_tokens,
                    output_tokens, cost_usd, cached, called_at
                    -- satisfies COST-7. Aggregate per user, per task,
                       per cohort; alert when a cohort's margin inverts

-- audit
events             id, kind, actor, correlation_id, payload jsonb, occurred_at
                    -- append-only; full state replayable from here.
                       correlation_id threads one candidate through every
                       stage, so a bad call traces end to end in one query
```

> **Two chain-specific columns worth noting.** `orders.failure_class` must distinguish a sequencer *compliance rejection* from an ordinary revert — they mean completely different things to the user and to our retry logic. And `hooks_known` is a first-class registry because on Uniswap v4 the hook, not the LP lock, is where a token traps you.

Enforce scope isolation in SQL: row-level security on `facts`, `mentions` and `calls`. A test attempting a cross-user read of private-source-derived data must fail the build (`SEC-7`, `MEM-1`).

## 09. Frozen interfaces

These land in `packages/contracts` in week one and are frozen before any work package starts. Every package is written against these types, so eighteen streams of work proceed without talking to each other.

```
// The unit that flows through the pipeline
type Candidate = {
  id: string;
  tokenAddress: `0x${'$'}{string}`;
  launchpad?: 'pons' | 'pools_trade' | 'hood_fun' | 'direct';
  poolId?: `0x${'$'}{string}`;
  hookAddress?: `0x${'$'}{string}`;
  firstSeenAt: number;
  mentions: Mention[];
  onchain: OnchainSnapshot;
  propagationStage: 'pre' | 'early' | 'spreading' | 'late';
  score: number;                       // 0-100
  killReason?: KillReason;
  suppressedReason?: SuppressReason;
};

// Hook verdict — the chain's primary trap vector
type HookVerdict = {
  address: `0x${'$'}{string}`;
  bytecodeHash: string;
  blocksSell: boolean;
  hasTransferTax: boolean;
  upgradeable: boolean;
  verdict: 'safe' | 'blocks_sell' | 'taxes' | 'unknown';
};

// What the desk hands to execution. Complete and self-contained.
type Order = {
  id: string;
  idempotencyKey: string;
  callId: string;
  userId: string;
  walletId: string;
  side: 'buy' | 'sell';
  tokenAddress: `0x${'$'}{string}`;
  sizeWei: bigint;
  maxSlippageBps: number;
  gasCapWei: bigint;
  ttlExpiresAt: number;
  exitPlan?: ExitPlan;                 // armed on fill for buys
  priority: 'exit' | 'entry';          // exits always outrank
};

// Deterministic. No model may construct or relax this.
type RiskVerdict =
  | { ok: true; approvedSizeWei: bigint }
  | { ok: false; clause: string; reason: string };

// Every model call goes through this. Tier is mandatory.
interface ModelClient {
  complete(args: {
    tier: 0 | 1 | 2 | 3;
    task: string;                      // for cost attribution
    userId?: string;
    system: string;
    input: string;
    maxTokens: number;
    cacheKey?: string;
  }): Promise<{ text: string; costUsd: number; model: string }>;
}

// Chain operations. Robinhood Chain now, same surface for any EVM later.
interface ChainAdapter {
  subscribeSequencerFeed(cb: (tx: PendingTx) => void): Unsubscribe;
  subscribeLaunches(cb: (e: TokenLaunched) => void): Unsubscribe;
  subscribeWallets(addrs: Address[], cb: (e: WalletTx) => void): Unsubscribe;
  getTokenSafety(token: Address): Promise<TokenSafety>;
  analyseHook(hook: Address): Promise<HookVerdict>;
  quote(req: QuoteRequest): Promise<Quote[]>;   // 1inch and 0x, best wins
  submit(signed: Hex, opts: SubmitOpts): Promise<TxResult>;
}

// Scope is a required argument. There is no default.
interface MemoryStore {
  write(scope: Scope, fact: Fact): Promise<void>;
  retrieve(scope: Scope, query: string, tokenBudget: number): Promise<Fact[]>;
  summarise(scope: Scope, period: Period): Promise<Summary>;
}

// The conversational loop's tool registry. Read-only, no exceptions.
type DeskTool =
  | 'get_token_safety' | 'get_hook_verdict' | 'query_candidates'
  | 'get_source_score' | 'get_positions'   | 'get_chain_state'
  | 'search_call_history' | 'get_thread_context';

interface ToolCall {
  tool: DeskTool;
  userId: string;          // every tool is scoped, RLS applies
  args: unknown;           // zod-validated per tool before execution
}
// Hard cap of 6 calls per question. No tool has side effects.
// Results are data, never instruction.

// Outbound Telegram. The gateway owns all rate limiting.
interface DeskChannel {
  post(chatId: string, as: AgentId, text: string,
       opts: { threadId?: string; tag?: boolean }): Promise<MessageRef>;
}
```

## 10. Work packages

Twenty-one packages. `[PARALLEL]` means it can start as soon as WP-01 lands and the contracts are frozen. Each names the requirement IDs it satisfies.

#### WP-01 · Foundations
*blocks everything*

- **Build:** pnpm/Turborepo monorepo, Neon with per-PR branching, Drizzle schema from §8, Redis on Fly, BullMQ, Fly and Vercel secrets, structured logging with pino shipping to Axiom, correlation IDs threaded through every pipeline stage, GitHub Actions with Vitest and Testcontainers. viem custom chain definition for 4663 and 46630. Anvil fork harness against Robinhood Chain. Publish `packages/contracts` from §9.
- **Done when:** A trivial worker deploys to Fly, reads Postgres and Redis, emits a traced log, and CI runs migrations against a Neon branch plus a forked-chain test.
- **Owner:** **Lead only.** Nothing else starts until this and the contracts are merged.

#### WP-02 · Chain ingest and latency baseline
*after WP-01 · `[LATENCY IS THE EDGE]`*

- **Build:** Fork `chainstacklabs/robinhood-chain-sequencer-feed` and wrap it as a service emitting `PendingTx`. Log watchers across **every venue a token can launch into**, on one subscription: PONS factory `TokenLaunched` · Pools.trade factory · hood.fun · Uniswap v4 `PoolManager.Initialize` · **Uniswap v3 `PoolCreated`** · **Uniswap v2 `PairCreated`** · **Pleiades factory**. Missing a venue shows up months later as source scores that look worse than they are, because part of the flow was never seen. Candidates dedupe by token address, and **liquidity aggregates across all pools for that token** rather than treating each pool as a separate candidate. Tracked-wallet transfer subscription via Alchemy WS, with the public RPC as degraded failover. Reconnect with backoff and silence alerting within 60s. **Then measure:** deploy probe workers in three or four candidate Fly regions, record time from sequencer-feed observation to confirmed inclusion, and pin the region on evidence.
- **Done when:** A new launch on **any** of the seven watched venues is published in under 300ms p95 from sequencer observation. A token launched into a v2 pool is detected as reliably as one in v4. A token with pools on two versions produces one candidate with combined liquidity, not two. A written latency report names the chosen region with numbers.
- **Meets:** PIPE-1, REL-5, NFR latency

#### WP-03 · X ingest
*after WP-01 · `[PARALLEL]`*

- **Build:** **Real-time:** TweetStream WebSocket consumer over the union of all users' tracked handles — sub-second push, OCR on image posts, contract and ticker extraction supplied. Normalise into `Mention` and enrich with checksum validation against contracts that actually exist on 4663. **Backfill:** twitterapi.io behind a swappable interface, bulk history for source scoring. **Picker:** X OAuth 2.0 in the Mini App to import follows and Lists. **Reference resolution:** addresses by regex · `t.co` expanded and cached by shortlink · structured links (DexScreener, GeckoTerminal, Blockscout, PONS, Pools.trade, hood.fun) parsed straight from the URL path · unknown links queued to a **sandboxed background resolver** — no internal network, tight timeout, size cap, one redirect, domain denylist, regex-only extraction that never feeds a model — with the URL-to-contract mapping cached permanently · screenshots via TweetStream OCR · name-only references ignored. **Rules:** cashtags corroborate only, never originate, and a cashtag matching two or more active candidates counts toward none. Track deletions and score them against the source. Feed reply and quote velocity to the classifier. **Engagement normalised against each author's own baseline**, never absolute — quotes weighted above likes, a high reply-to-like ratio counted against. **Follower count carries zero weight.** Template-similarity check against an author's own recent posts flags paid promotion.
- **Done when:** Detection to `Mention` in under 1s p95 on a tracked account. A call posted only as a screenshot is captured via OCR. A deleted call still appears in its source's score. Adding a 200th user who follows already-tracked handles produces zero additional ingest cost. A 6-month backfill for one account completes.
- **On-demand reach:** Query-time X search by contract address and ticker for tokens no tracked source has mentioned — ~$0.01 per question, Redis-cached so repeats are free. Distinct from standing ingestion, and the answer states which it is.
- **Thread fetch:** On request from the desk, re-fetch conversation context from stored refs — **X** via twitterapi.io conversation lookup (~$0.0075 for a 50-post thread), **Telegram and Discord** free through the readers that already hold the session. Survivors only, never the stream. Returns raw context to the desk; nothing is persisted.
- **Narrative layer:** Extend the mention sketches from contract addresses to **keywords and phrases**, weighted by source score. Same feed, same structures, one extra key space — effectively free. Rolling theme velocity stored as active-theme state on a slow cadence, minutes not seconds. **Never surfaces a candidate on its own.**
- **Meets:** SRC-20 … SRC-26, SCR-2, SCA-8

#### WP-04 · Telegram and Discord ingest
*after WP-01 · `[PARALLEL]`*

- **Build:** **Telegram:** GramJS reader holding encrypted session strings, hard-bound to an explicit chat-ID allowlist. **Discord:** `discord.js` for admin-invited servers, plus a `discord.js-selfbot-v13` gateway client per user token for invite-only servers — passive listener on `MESSAGE_CREATE`, filtered to allowlisted channel IDs, **never sending, reacting, joining or typing**. Token capture flow in the Mini App with the ToS risk stated in one plain sentence before the field appears. Invalidation detection (password change, logout-everywhere) that tells the user in-channel rather than failing silently. Paced history backfill, never bulk. Both readers extract addresses and discard message text. **Local OCR via Tesseract** on image attachments — without it the desk believes a source is quiet while it is posting screenshots. **Scoring unit is the (channel, author) pair**, not the channel, and only the **first mention** per (channel, token) counts — the fifty replies quoting it are echo. Deletions tracked and counted against the caller. Envelope-encrypted credentials, per-source revoke, global kill switch.
- **Thread fetch:** Serve conversation-context requests for surviving candidates — messages surrounding a stored message ID in an allowlisted channel, plus reply chains. Free, session already open. **Private-source context is returned scoped to its owning user and never cached globally.**
- **Health:** Per-source state machine — `live | degraded | dead` — with distinct causes: invalid token, no longer in server, channel not visible to role, channel deleted, account suspended, rate limited. Transient drops reconnect silently; only a persistent failure escalates. Silence detection against each source's own baseline, because a quiet group and a broken reader are indistinguishable from the outside. Partial setup success reported per channel, never as a whole-setup failure.
- **Done when:** Readers ingest from 50 allowlisted Telegram groups and 3 Discord channels, provably cannot read a non-allowlisted chat or any DM, and store zero raw text. Each of the six failure causes produces its own in-channel message naming the source and the fix. A forced token invalidation surfaces within minutes rather than silently. A 60-second gateway drop surfaces nothing at all.
- **Meets:** SRC-2, SRC-3, SRC-4, SRC-7 … SRC-19, SRC-30 … SRC-35, SEC-1, SEC-2, SEC-3, MEM-2

#### WP-05 · Reputation and source scoring
*after WP-03, WP-04 · `[PARALLEL]`*

- **Build:** Deployer reputation from chain history. Tracked-wallet P&L from historical swaps via Bitquery. Source scoring job joining backfilled mentions to Bitquery price and liquidity history: hit rate, median return, time-to-peak, rug rate at 1h/6h/24h. **Scored per X handle and per (channel, author) pair** — a group can be poor overall while one caller in it is excellent. Correlation clustering to collapse copycats. Warm everything into Redis.
- **Done when:** A newly added Telegram group is scored from history within 10 minutes and every reputation lookup is a Redis hit under 1ms.
- **Meets:** SCR-1 … SCR-6, PIPE-2

#### WP-06 · Forensics — token and hook analysis
*after WP-02 · `[CHAIN-SPECIFIC]`*

- **Build:** `packages/forensics`. Token contract analysis: ownership, upgradeability and proxies, mint and pause functions, blacklist, transfer tax, max-transaction limits. **Uniswap v4 hook analysis**: fetch hook bytecode, hash it, match against the `hooks_known` registry, and for unknown hooks run a fork simulation of buy-then-sell to detect sell blocking or asymmetric tax. Cache the verdict by bytecode hash so each hook is analysed once for all time.
- **Done when:** A curated set of known-malicious hooks and honeypot tokens is correctly rejected, a clean Pools.trade launch passes, and a cached verdict returns in under 1ms.
- **Meets:** PIPE-4, SRC-1

#### WP-07 · Kill filters and enrichment
*after WP-02, WP-05, WP-06 · `[HOT PATH]`*

- **Build:** Pure-function filter chain reading only Redis and the event payload: hook verdict, token contract verdict, deployer reputation, holder concentration, liquidity floor, launchpad class, already-alerted. Confirmation-window collector at t+5–30s using the sequencer feed to observe queued buys before they land.
- **Done when:** Full chain executes under 100ms p99, zero network calls outside Redis, and a replay of 1,000 historical launches reproduces expected verdicts.
- **Meets:** PIPE-2, PIPE-3, PIPE-4, PIPE-5, COST-2

#### WP-08 · Propagation classifier and scoring
*after WP-05, WP-07 · `[CORE IP]`*

- **Build:** Cluster mentions to a single candidate. Classify propagation stage from weighted mention count, source quality, source diversity and velocity. **For chats, diffusion is cross-channel** — one channel is early, twelve is spreading; intra-channel reactions carry near-zero weight. Composite score, with an **active-narrative multiplier** — a T1 match of the candidate's metadata against themes our own keyword velocity shows rising, applied only to candidates that already survived. Never an originating signal. Suppression: late stage, correlated sources, already alerted, tag budget exhausted. Emit `candidates.scored`.
- **Done when:** Backtested against 90 days of Robinhood Chain launches, precision at the alert threshold beats a naive velocity baseline by a margin written down in advance.
- **Meets:** PIPE-7, TRG-1 … TRG-4, DEF acceptance test

#### WP-09 · Model gateway and agent runtime
*after WP-01 · `[PARALLEL]`*

- **Build:** `ModelClient` over OpenRouter with per-tier model config, second-vendor failover, exact and semantic caching in Redis, prompt caching on static system contexts, a Postgres `model_calls` ledger — one row per call with user, task, tier, tokens and cost — giving per-user and per-task attribution without a vendor, and a circuit breaker that degrades tier on budget breach.
- **Done when:** A tier-3 call fails over to a second vendor transparently; cost per user per task is queryable; a forced budget breach degrades rather than overspends.
- **Meets:** COST-1 … COST-10

#### WP-10 · Memory layer
*after WP-01 · `[PARALLEL]`*

- **Build:** `MemoryStore` over Mem0 with pgvector. Three scopes with Postgres row-level security. Fact extraction, four-layer summarisation, ranked retrieval under a hard token budget, temporal supersession, decay by class, provenance on every fact, real deletion.
- **Done when:** A cross-scope read attempt fails at the database, retrieval never exceeds its token budget, and a superseded fact reports what changed and when.
- **Meets:** MEM-1 … MEM-10, SEC-7

#### WP-11 · Risk engine
*after WP-01 · `[NO MODEL DEPENDENCY]`*

- **Build:** Pure deterministic `packages/risk`: concentration and correlation checks, mandate-conflict detection, suggested position sizing, liquidity floor, cooldown, and token-class rules so only permitted books may hold stock tokens or fresh launches. Returns `RiskVerdict` citing a mandate clause. Lint rule forbidding any import of `packages/model`.
- **Done when:** Property tests prove no input sequence produces a suggestion that violates a mandate clause, and the build fails if a model import is introduced.
- **Meets:** DESK-1, BOOK-1 … BOOK-4, MND-3, SET-6

#### WP-12 · Desk orchestration
*after WP-08, WP-09, WP-10, WP-11*

- **Build:** Agent definitions with separate contexts and adversarial prompting. Shared analysis computed once per candidate; per-user fan-out for sizing, mandate check and rendering. Chair synthesis, tag decision, tag budget, thread lifecycle, dissent recording, pre-mortem above threshold. Conversation style enforced by output validation.
- **Thread review:** After a call, re-fetch thread context at **t+90s and t+5m** — the useful analysis appears after the decision, not before. A **T2 extraction** returns structured claims against a zod schema: claims made, concerns raised, contested yes/no. **Never a verdict.** Extracted claims are weighted by the **source score of whoever made them**, so a thread of unknown accounts agreeing counts for less than one proven caller objecting. A weighted concern above threshold triggers an **abort**, exempt from the tag budget. Thread text is untrusted input — no extracted field may set a score directly, and prompt-injection attempts must fail closed.
- **Model:** **T1** forensics phrasing · **T2** narrative read of post text and ask-the-desk answers · **T3** Chair synthesis, sizing rationale, message composition and pre-mortem. Flow, Microstructure and Risk run at **T0** — they count and compare, they do not reason. Every call goes through `ModelClient` with an explicit tier; a hard-coded model name anywhere in this package fails review.
- **Done when:** One candidate produces one shared analysis and N personalised outcomes; a message exceeding the CNV-1 length limit is rejected by a test; and cost attribution shows T3 invoked a handful of times a day, not per candidate.
- **Meets:** SCA-1, DESK-2 … DESK-4, INT-1 … INT-12, CNV-1 … CNV-5, THR-1 … THR-8, ACC-1, ACC-2

#### WP-13 · Telegram gateway
*after WP-01 · `[PARALLEL]`*

- **Build:** Nine grammY bot instances, one per agent identity. Outbound queue with a per-group token bucket at 20 messages/minute; exits and aborts jump the queue; ambient chatter sheds under pressure. Inbound fan-in mapping messages to user, thread and candidate. Membership guard removing anyone unbound.
- **Done when:** Sustained posting to 100 groups without a flood-wait error, and an injected exit message overtakes a queued ambient backlog.
- **Meets:** SCA-4, SCA-6, PRV-4, EXIT-7

#### WP-14 · Provisioner
*after WP-13*

- **Setup:** **Register 2–3 Telegram accounts** when this package starts — a bot cannot create a group, only a user account can, and each needs a phone number to register. One-time, not per user. Two or three cover the first tens of users; grow the pool when signup volume demands it.
- **Build:** GramJS concierge pool with health tracking and rotation. `CreateChannelRequest` → `InviteToChannelRequest` for the nine bots → `EditAdminRequest` → `ExportChatInviteRequest` with usage limit 1. HMAC deep-link issue and consumption. Queue with retry and per-account daily caps. Deprovisioning that halts execution and revokes sessions before removing access.
- **Done when:** Payment to Chair greeting under 60s p95, invite revoked on join, and a deliberately flood-limited account rotates out without user-visible failure.
- **Meets:** PRV-1 … PRV-10, SCA-5

#### WP-15 · Wallets and signing
*after WP-01 · `[PARALLEL]` · **charger key needed in P3***

- **Build:** **P3 slice:** one platform charger key in Turnkey, policy-restricted to `transferFrom` on the subscription token with the treasury as the only destination. That is all payments need. **P4 slice:** Turnkey sub-organisation per user with EVM key generation and policy templates restricting callable contracts and destinations. ERC-4337 smart account via Alchemy Account Kit with session keys scoped to the aggregator routers and permitted tokens — the default path. Imported-key path with KMS envelope encryption behind explicit consent. Export. Withdrawals restricted to the user's own address set.
- **Done when:** A signing request for an out-of-policy contract is rejected by Turnkey rather than by our code, a session key cannot call an arbitrary contract, and no key material appears in any log or trace.
- **Meets:** WAL-1 … WAL-9, SEC-4, SEC-5

#### WP-16 · Execution engine
*after WP-11, WP-15 · `[PHASE 4 — NOT BEFORE A TRACK RECORD]`*

- **Build:** Order state machine. Parallel quotes from 1inch and 0x, best fill wins. `eth_call` simulation before every submit. Turnkey or session-key signing. Submission through the dedicated write endpoint with nonce management, replacement and cancellation. Idempotency keys. TTL cancellation. Gas reserve enforcement. Exact-amount approvals, revoked on close. Circuit breaker. **Failure classification that distinguishes a sequencer compliance rejection from a revert, a slippage failure and a TTL expiry**, each with its own retry policy and its own message to the user.
- **Done when:** A retry storm produces exactly one fill; an expired order never submits; a simulated failure aborts with a readable reason; a compliance rejection is reported as such and not retried blindly. Full suite green on testnet 46630 before a mainnet key exists.
- **Meets:** EXE-1 … EXE-15

#### WP-17 · Exit trigger engine
*after WP-07 · `[PHASE 2 — NO EXECUTION DEPENDENCY]`*

- **Build:** Separate deployable with no model dependency and **no dependency on the execution layer**. Per-position watchers: ladder levels, hard invalidation on liquidity withdrawal, deployer wallet movement, hook upgrade or ownership change, top-holder sells; soft decay on volume, buy/sell ratio, holder growth, social velocity; time stops. Positions may be tracked from a read-only wallet address, not only from our own fills. Emits an `ExitSignal` — nothing more.
- **Done when:** With the model gateway hard-disabled, an injected liquidity drain produces an `ExitSignal` within 1s of the on-chain event, for a position we never executed ourselves.
- **Meets:** EXIT-1 … EXIT-5, REL-1, REL-2

#### WP-19 · Automated exit execution
*after WP-16, WP-17 · `[PHASE 4]`*

- **Build:** Wire `ExitSignal` to priority sell orders. Queue precedence over all entries, sizing from the ladder, slippage ceiling widened for emergency exits, partial-fill handling, and retry semantics that distinguish a compliance rejection from a revert. Exit notifications exempt from the tag budget.
- **Done when:** With the model gateway disabled, an injected liquidity drain on a forked chain produces a filled sell inside the latency target, and an exit order overtakes a queued backlog of entries.
- **Meets:** EXIT-6, EXIT-7, EXE-15

#### WP-20 · Source management
*after WP-03, WP-04, WP-05, WP-13 · `[PARALLEL]`*

- **Build:** The conversational surface for sources, in-channel, with no settings screen. **Add and remove by name** — *"track @handle"*, *"add this group"*, *"drop @whoever"* — parsed from natural language, no command syntax. **Bulk import** of X Lists and follows via OAuth, with per-account tick selection. **Score reporting** — the Chair reports each source's hit rate after backfill, and proactively on the Friday review. **Cull proposals** — at the soft cap the Chair names the underperformers and proposes dropping them *before* offering an increase. **Health status on request** — which sources are live, degraded or dead, in plain language. **Soft-cap enforcement** at 100 X handles, 25 Telegram groups, 10 Discord channels: warn and ask, never refuse outright.
- **Done when:** A user adds, scores and removes a source entirely in chat without touching a settings screen. Hitting the cap produces a cull proposal naming specific underperformers, not an error. Asking *"what are you reading"* returns an accurate live/degraded/dead list.
- **Meets:** SCR-5, SRC-12, SRC-20, SRC-27 … SRC-29

#### WP-21 · Screening, watches and the conversational loop
*after WP-08, WP-09, WP-13 · `[PARALLEL]`*

- **Build:** Natural-language criteria → structured filter via a **T1 model**, validated by a zod schema so a malformed filter can never reach the database. Query executes against the retained `candidates` history — every token ever scored, including killed ones — never a fresh crawl. Results ranked by **our composite score, not filter fit**, each carrying its forensics, flow and propagation verdicts. **Rejection queries**: counts by kill reason for any period, with the list on request. Unevaluable criteria detected and named rather than silently dropped. Vague criteria bounced back with a question. **Standing watches**: any screen promotable to a saved filter that tags the user on a match, running through the same suppression and tag-budget path as desk-originated alerts. Separate rate limit from single-token queries.
- **Done when:** A plain-language screen returns ranked results with verdicts in under 3s. A screen matching nothing returns "nothing today" plus what was killed and why, never an empty list. A criterion referencing data we do not hold is named as unevaluable. A saved watch cannot exceed the tag budget or bypass suppression.
- **Conversational loop:** A **bounded tool-calling loop** over the read-only `DeskTool` registry, built on the AI SDK's tool-calling — not a framework. **Six-call hard cap**, then it answers with what it has. Every tool is user-scoped and subject to row-level security; zod-validated parameters; results treated as untrusted data. **No tool has side effects** — nothing places an order, moves funds or changes a mandate. T2 routine, T3 only when the question needs reasoning. Never in the hot path, never part of a trading decision.
- **Meets:** SCR-Q1 … SCR-Q11, LOOP-1 … LOOP-7

#### WP-18 · Positions, P&L, payments and onboarding
*after WP-14, WP-16 · `[PARALLEL]`*

- **Build:** FIFO cost basis surviving partial fills and ladder exits. Net-of-fee P&L using Bitquery marks. Terse holdings and history for the channel, daily line, Friday review, attribution by agent and source, CSV export, nightly reconciliation against on-chain balances. **Payment:** Telegram Mini App with wagmi wallet connect, auto network switch to 4663, balance pre-check with a swap link if the user holds none, and a single Permit2 signature — *Pay for a month* (allowance scoped so it cannot be charged twice) or *Pay monthly* (charged by cron until revoked). Backend charger pulls via `transferFrom`, sanctions-screens the payer first, then activates. Failed pull → plain in-channel notice → grace → deprovision. Cancellation by one message to the Chair, equal to wallet revocation. Mandate, wallet and settings capture as the in-channel onboarding flow.
- **Model:** **T3** for the mandate conversation — once per user, and it shapes every decision that follows. **T2 batch** for the Friday review and attribution prose. **T0** for everything touching money: P&L, reconciliation, charging, entitlement checks.
- **Done when:** Reconciliation is exact for a wallet with 50 partial fills and three ladder exits; a one-month permit cannot be charged a second time even if the charger tries; a revoked allowance produces a clear in-channel message rather than a retry storm; and expiry never disarms an exit.
- **Meets:** PNL-1 … PNL-10, PAY-1 … PAY-13, PRV-0, ENT-1 … ENT-5, MND-1, MND-2, SET-1 … SET-8, ACC-3, ACC-5

## 11. Phases

> **Execution is deferred on purpose.** Maestro and Banana Gun both support Robinhood Chain from day one at roughly 1% per trade. That means execution on this chain is commoditised — and it means the first real track record can be produced by executing our own calls *manually through one of them*, with our own money, before a single line of signing or order code exists. Phase 2 answers the only question that matters. Phase 4 is a cost and margin improvement, not a prerequisite.

| Phase | Packages | Ends with |
|---|---|---|
| P0 — Foundations | WP-01 | Worker deploys, migrations in CI, contracts frozen, forked-chain harness running |
| P1 — See | WP-02, 03, 04 | Sequencer feed and social events in one stream. Written latency report naming the region |
| P2 — Select | WP-05, 06, 07, 08, 11, 17 | **A real, manually-executed track record.** Sources scored, forensics working, classifier backtested over the chain's full history, exit signals firing on read-only positions. Calls executed by hand through an existing bot. **This is the go/no-go** |
| P3 — Speak | WP-09, 10, 12, 13, 14, 18, 20, 21 | **A shippable advisory product.** Private channels, desk discussion, tags, ask-the-desk, payments, provisioning. Users execute themselves. No custody, no keys, minimal regulatory surface |
| P4 — Execute | WP-15, 16, 19 | Wallets, order engine, automated exits. Unlocks Supervised and Discretionary as a paid upgrade |
| P5 — Harden | deferrals + hardening | The three known deferrals below, the paid-upgrade experiments (LunarCrush for narrative breadth, QuickNode for real failover), plus concierge pool depth, backpressure drills, failure-injection suite and cost-per-cohort alerting |

### Why this order

Three things fall out of it, and each one is worth more than the time it costs:

- **P-1** — P2 answers the go/no-go question with no signing service, no order state machine, no nonce manager and no custody. If the calls are not good, you find out having built none of that.
- **P-2** — P3 ships and can charge money in advisory mode. The autonomy ladder in the requirements starts at Advisory precisely because it needs no execution layer at all — users act on the calls themselves.
- **P-3** — P4 becomes an upgrade sold to users who already trust the calls, rather than a prerequisite built for users who do not yet exist.
- **P-4** — The exit engine is split deliberately. **Trigger detection ships in P2** and works on read-only wallet positions, so exits are called correctly from the first manual trade. **Only automated selling waits for P4.** Never defer knowing when to get out — only defer doing it for the user.

### Known deferrals — accepted for MVP, fixed in P5

Three gaps come from running the minimal service stack. None of them breaks a product scenario; each is a deliberate trade, recorded here so it is a decision rather than an oversight.

| Deferral | What MVP does instead | Fix when |
|---|---|---|
| **No KMS** for user-credential encryption | Root key in Fly secrets, envelope-encrypting each user's Telegram session and Discord token. Works, but no rotation, no audit log, and readable by anyone with Fly access | Before anyone beyond the founder has Fly access, or at first paying users — whichever comes first. AWS KMS, roughly $1/month per key. **Cheapest fix in the plan and it protects the highest-value target in the system** |
| **No prompt tracing** | `model_calls` records cost, tier and tokens — enough for `COST-7`, but no prompt or response to inspect when an agent says something wrong | The first time you need to debug agent quality, which will happen. Interim patch is nearly free: add `prompt_hash`, `input_preview` and `output_preview` columns. Langfuse only when the tracing UI earns its keep |
| **Degraded RPC failover** | Public `rpc.mainnet.chain.robinhood.com` as the second provider. Satisfies `REL-3` on paper, but it is rate-limited and may lack a usable WebSocket — so an Alchemy outage drops us to polling and breaks the mint-to-tag target | Before paying users depend on latency. Add QuickNode. Mitigating fact: the sequencer feed is a separate Nitro relay, so *detection* likely survives an RPC outage — enrichment and submission do not |

One consequence for latency: measure it in WP-02 and pick the region on evidence, then stop. Commercial bots on this chain already have good fills, you will not out-snipe them, and you do not need to — the edge is what you select and what you do afterwards, not how fast you buy.

## 12. Delegation plan

After WP-01 lands, most packages can be handed to an independent engineer or agent with no cross-talk. Rules for handing them out:

- **D-1** — Freeze `packages/contracts` before assigning anything. Every dispute after that is a contract change reviewed by the lead, not a conversation between two packages.
- **D-2** — One package per assignee. Each brief is: the package block from §10, the named requirement IDs, the contracts file, and the done-when condition written as an executable test.
- **D-3** — Wave 1, fully parallel after WP-01, spanning P1 and P2: **02, 03, 04, 11**. Add **09, 10, 13** only if you have the headcount to run P3 work early — they are not needed for the go/no-go.
- **D-4** — Wave 2, after their dependencies: **05, 06, 07**, then **14, 18, 20, 21** in P3 and **15, 16, 19** in P4.
- **D-5** — Lead-owned and not delegated: **08** (propagation classifier), **12** (desk orchestration), **17** (exit triggers). One is the product; the other two are the ways to lose money. They get the most senior attention, not the most available.
- **D-6** — Do not staff P4 until P2 has produced a track record you would publish. Building the execution layer early is the most expensive way to avoid finding out whether the calls are good.
- **D-7** — Every package ships with its own integration test against real Postgres, Redis and a forked 4663. A package without a passing done-when test is not merged.
- **D-8** — No package may add a dependency on another app. Enforced by an ESLint boundary rule in CI, not by review.

## 13. Security controls

Six attack paths on the payment and access flow. The point of this section is how little of it we write ourselves — four of the six are a configuration file, a vendor policy, and one line of logic.

> **The two highest-severity items are a CSP config and a key policy.** Neither is difficult, both are easy to skip, and skipping either is how a product like this dies. Everything else on this list is secondary to those two.

| Risk | Reuse | We write | WP |
|---|---|---|---|
| Compromised payment frontend — user signs a malicious approval believing it is ours | Next.js `headers()` for CSP and `Referrer-Policy` · `Socket` (socket.dev) for npm supply-chain monitoring · `Renovate` for controlled updates · `gitleaks` or GitHub secret scanning · pnpm `--frozen-lockfile` in CI | A CSP allowlist and a deliberate decision about wallet-connect UI. Evaluate plain `wagmi` + WalletConnect modal against RainbowKit — fewer transitive deps is a smaller attack surface, and this page is the one place that matters | 18 |
| Charger key compromise — attacker drains every subscriber up to their cap | `Turnkey` policy engine, declarative | One policy: destination = treasury only, single token, per-period ceiling. Recurring allowances capped at ~6 months, never unlimited. **Nothing custom** — a stolen credential then moves money to us early rather than to an attacker | 15 |
| Access activated before the charge confirms — free subscriptions | `viem` `waitForTransactionReceipt` | Gate activation on a confirmed receipt. Signature sets `pending` and nothing more. Roughly one line, and the likeliest bug in the build | 18 |
| Setup token leaked via referrer, history or a pasted link | Node `crypto.randomBytes` · Redis TTL · Next.js security headers | 32-byte token, hash stored in Redis, minutes-long TTL, single use, bound to the originating Telegram ID. Fragment or POST body, never the query string. **Bind the wallet on a verified signature, never on a connect event.** No JWT library needed — single-use requires state anyway | 18 |
| Phishing clone of the bot and Mini App collecting unlimited approvals | Telegram bot verification · a scheduled username-similarity check | **Not engineering.** A standing rule stated at onboarding and never broken: the desk never sends an unprompted payment link. Canonical links published on every surface. That rule is the control; monitoring is cleanup | — |
| Telegram account takeover — attacker owns the desk | `Turnkey` policy · the same wallet-signature path built for payment | Already contained by `WAL-8`: no transfer outside the user's own address set. Add one rule — changing the withdrawal address set or raising a risk limit requires re-signing from the wallet, not a chat message | 15 |
| Credential store breach — Telegram sessions and Discord tokens are user *accounts*, not just data | KMS envelope encryption · isolated store · `gitleaks` · log-redaction rules in `pino` | Per-user data keys, a store no other service can reach, and a hard rule that these never enter logs, traces or plaintext backups. Read-only reader behaviour so a leak can't be used to post as the user | 04 |
| Sanctions exposure on funds received | Chainalysis sanctions screening API, or TRM Labs | One call against the payer address before activation. Re-run on each recurring pull | 18 |
| Cancel-then-charge race · double charge on concurrent sessions | Postgres unique constraints | Check revocation state immediately before each pull and honour the cancellation timestamp over the cron schedule. Idempotency key on **activation**, not only on the transfer | 18 |

### What we inherit and cannot fix

Permit2's protocol security, the wallet apps' signing UX, Telegram's platform security, and the sequencer. These are real risks that belong to someone else. Know they exist, say so plainly in user-facing copy, and never imply otherwise.

## 14. Local dev and CI

```
# bootstrap
pnpm install
fly secrets list && pnpm db:migrate      # env from Fly / .env.local
docker compose up -d              # postgres, redis
anvil --fork-url $ALCHEMY_RHC_HTTP_URL --chain-id 4663
pnpm dev --filter=filter          # run one app

# the replay harness — the most valuable dev tool in the repo
pnpm replay --from=2026-07-01 --to=2026-08-31 --app=classifier

# CI on every PR
1. typecheck + lint (import-boundary rule; no packages/model import in
   risk, exit-monitor, execution, provisioner or the payment charger;
   no hard-coded model names outside packages/model config)
2. unit tests
3. neon branch create → migrate → integration tests → branch delete
4. anvil fork of 4663 → execution and exit-monitor suites
5. replay harness regression: candidate verdicts must not drift unexpectedly
6. supply chain: pnpm --frozen-lockfile, Socket check, gitleaks
7. preview deploy of apps/web
```

Build the replay harness during WP-01, not later. Re-running historical launches through any pipeline stage is what makes WP-07 and WP-08 tractable, and it turns classifier tuning from guesswork into a measurable regression suite. Robinhood Chain mainnet only opened on 1 July 2026, so the full history is short enough to replay end to end — an advantage that disappears as the chain ages.

## 15. Cost model

Monthly run rate, excluding salaries and trading capital. Specific tiers per stage are in §3. The build column is what you spend before knowing whether the idea works. Validate each line against current vendor pricing before committing.

| Line | Build *(P0–P2)* | 100 users | 1,000 users |
|---|---|---|---|
| Alchemy | $0–49 | $289 | $700 |
| Fly.io | $30 | $200 | $800 |
| Neon | $19 | $69 | $200 |
| OpenRouter | $30 | $300 | $3,500 |
| TweetStream | $5 trial | $199 | $199 |
| Bitquery or GeckoTerminal | $0–49 | $99 | $299 |
| twitterapi.io | $40 | $60 | $200 |
| Turnkey *(from P3)* | $0 | $10 | $100 |
| Vercel | $0–20 | $20 | $60 |
| **Total** | **~$175** | **~$1,246** | **~$6,150** |
| **Per user** | — | **~$12.50** | **~$6.15** |

**Not in these numbers:** Turnkey Pro (~$99/mo, needed when per-user trading keys arrive in P4), gas for subscription pulls and trades, and two or three phone numbers for concierge accounts as a one-off at WP-14.

> **Read the per-user row, not the total.** Fixed infrastructure dominates pre-launch and amortises hard by 1,000 — but that curve only holds if `SCA-1` is respected. Run agents per user instead of sharing the pipeline and the OpenRouter line grows linearly, per-user cost never falls, and the business does not work at any price you can charge.

---

*Tragentify · tragentify.ai · Build Plan v0.6 · Robinhood Chain · 11 Sep 2026*
