# Tragentify

*Tragentify · tragentify.ai · Product Requirements v0.3*

Your private team of agents, making the best trades on your rules — instead of you wading through the noise across call groups and crypto Twitter.

**Status** Ready to build · **Date** 11 Sep 2026 · **Scope** MVP through Book C · **Owner** Founder

---

## 01. Product definition

Forty Telegram groups, hundreds of accounts on X, paid Discords, launches every minute, threads where the useful part is the fourteenth reply, screenshots instead of addresses. Keeping up with it is a full-time job and people still lose — they miss the one that mattered because they were asleep, buy the loud one because it is the one they saw, size wrong, and never write down why they bought it.

Tragentify is **your private team of agents, making the best trades on your rules** — instead of you wading through the noise across call groups and crypto Twitter. Ask them anything, or let them bring things to you. They have the whole picture; you set the rules they work to.

> **The team is the product.** Not a feed, not a filter, not a signals service. Every capability in this document exists to make that team worth having.

### How it stays early

A mechanism, not the product. Every token spreads along a curve — a few sources, then many, then everyone. The desk measures where a token sits on that curve and speaks while it is still early, staying silent once something is already everywhere. That is what lets it be useful in the window where acting still pays, rather than confirming what a user has already seen five times.

### Acceptance test

One question for any proposed feature: **does it make the team more useful to have?** Either it answers something a user would otherwise go hunting for, or it brings them something worth knowing. A feature that does neither is not part of this product, however interesting it is.

- **DEF-1** — One private channel per user. Never a shared broadcast channel.
- **DEF-2** — Every recommendation is weighed against that user's mandate, stated risk appetite and existing exposure. Generic calls broadcast identically to everyone are prohibited.
- **DEF-3** — The desk must be capable of saying nothing for a full day, and must report that it did.
- **DEF-4** — The desk must be capable of refusing the user, including naming behavioural patterns such as revenge trading.
- **DEF-5** — Every opened position is closed out in the channel with an outcome, including losses. `[HARD]`

## 02. Books

Three books share one desk but run separate clocks and evidence standards. They are **coverage universes, not capital allocations** — the desk advises on each; how much of anything a user holds is theirs alone to decide.

| Book | Universe | Clock | Evidence | Posture |
|---|---|---|---|---|
| A — Core | Majors, liquid L1/L2 | Days–weeks | Macro, flows, positioning, funding, ETF and CEX data | Patient. Few calls, longer horizons |
| B — Rotation | Midcaps, narratives | Hours–days | Unlocks, revenue, narrative rotation, relative strength | Selective. Narrative and rotation driven |
| C — Origination | Tokens minted minutes ago | Minutes–hours | Deployer history, holder distribution, smart-money entry, liquidity mechanics | Aggressive but rare. Most candidates are killed |

- **BOOK-1** — The desk sets **no capital limits and holds no budget**. It advises; the user decides what to commit. Any sizing the Chair states is a recommendation, never an allocation drawn from a pool. `[HARD]`
- **BOOK-2** — Each book carries its own response window, evidence standard and suggested sizing style.
- **BOOK-4** — Concentration and correlation are assessed across books, not within them — a user already heavy in one sector hears about it regardless of which book the call came from.

## 03. The desk roster

Each agent owns a distinct real-time data source. Agents differentiated only by personality are prohibited — they read the same context and manufacture agreement.

| Agent | Owns | Fires on | May tag user |
|---|---|---|---|
| Flow | Tracked wallets, CEX in/outflows, whale movement | Tracked wallet moves size | No |
| Microstructure | Liquidity depth, spreads, funding, open interest | Funding spike, liquidity drain | No |
| Narrative | Social velocity, source mentions, propagation stage | Velocity break, confluence | No |
| Fundamentals | Tokenomics, unlocks, revenue, treasury | Unlock calendar, protocol event | No |
| Forensics | Contract risk, deployer history, LP state, honeypot sim | Any new candidate | No |
| Macro | Rates, DXY, equity correlation, ETF flows | Scheduled prints, regime shift | No |
| Risk | Concentration, correlation, mandate conflicts, quality gates | Continuous | No — holds veto |
| Chair | Synthesis, sizing, thesis authorship, user contact | Always | Yes — sole |
| Execution | Wallets, order routing, fills, gas, position accounting | Order from Chair only | No |

- **DESK-1** — Risk holds a veto enforced in deterministic code. Position size, concentration, drawdown and liquidity floors are checked before an order is constructed. No LLM may relax a limit. `[HARD]`
- **DESK-2** — Only the Chair may tag the user. Specialists post findings and cannot escalate.
- **DESK-3** — Agents run on separate contexts with adversarial prompting. Convergence is a defect — a bear agent that never dissents is a failing component.
- **DESK-4** — Above a size threshold, one agent is assigned to argue the position to zero before entry.

## 04. Mandate and autonomy

Onboarding produces a written Investment Policy Statement, not a settings screen.

- **MND-1** — The mandate is captured through conversation with the Chair and rendered as a document the user can read and edit.
- **MND-2** — Mandate captures: risk appetite per book, the universe they want covered, how aggressive to be on fresh launches, concentration preferences, time horizons, and explicit vetoes. **No capital figure and no spend limits** — the desk advises, it does not hold or allocate money.
- **MND-3** — Every decision cites the mandate clause it satisfies or violates. Refusals name the clause.
- **MND-4** — Three autonomy modes, set per book. *Advisory*: desk recommends, user executes. *Supervised*: desk executes within limits, escalates outside them. *Discretionary*: desk executes and reports.
- **MND-5** — A drawdown breach automatically de-risks and downgrades autonomy one level until the user re-authorises.
- **MND-6** — Overrides are permitted, must be explicitly confirmed, and are logged against outcomes.
- **MND-7** — The mandate is versioned. Every call records the version in force at the time.

## 05. Sources and access

### Three tiers

| Tier | Contents | Signal visible to |
|---|---|---|
| Base | Curated groups, servers and accounts we operate | All users |
| User public | Public X accounts, public Telegram groups | All users |
| User private | The user's paid or private groups | That user only |

> **Isolation rule.** Signal derived from a user's private source never reaches another user. Redistributing paid alpha gets us banned from every group and is an existential legal risk. Enforced in the data model, not in policy.

### Access method per platform

| Platform | Method | Constraint |
|---|---|---|
| Telegram — admin | User adds our bot, privacy mode disabled | Compliant and simple, but most call groups disallow bots |
| Telegram — member | Session delegation via Telethon or TDLib, allowlisted chat IDs | Only route to private groups. Highest-risk credential we hold |
| Discord — admin | Bot invite with Manage Server | Clean, but almost nobody is an admin of a call group they pay to be in |
| Discord — member | The user's own account token, gateway client, allowlisted channels | **The only path that reaches an invite-only group.** Grants exactly what their account can see. Violates Discord ToS — the user's account carries the ban risk and must be told once, plainly, before we ask for it |
| X — follows and Lists | OAuth 2.0, `tweet.read` + `users.read`, import and select | Best onboarding conversion. Build first |
| X — public handles | Paste | Trivial |
| On-chain | WebSocket subscription: pool creation, tracked wallets, liquidity events | Truth layer |

- **SRC-1** — Social sources produce *claims*. On-chain produces *truth*. No candidate reaches the user on a social mention alone — every social hit is verified on-chain first. `[HARD]`
- **SRC-2** — The Telegram reader operates on an explicit chat-ID allowlist enforced at the client. It must be structurally incapable of ingesting a non-allowlisted chat or any DM.
- **SRC-3** — The reader forwards extracted contract addresses plus minimal context. Raw message archives are never transmitted or stored.
- **SRC-4** — Per-source revoke, plus a global kill switch that invalidates the session immediately.
- **SRC-5** — Onboarding order: X OAuth, then Telegram, then Discord. Every step skippable — the base set alone must deliver value with zero user sources connected.
- **SRC-6** — Onboarding states plainly what the desk can see and what is never stored.
- **SRC-7** — Discord member access uses the user's own account token against the gateway. **Read-only always** — the reader never sends, reacts, joins or types, because write behaviour is what gets accounts flagged. `[HARD]`
- **SRC-8** — The ToS and ban risk is stated once, in one plain sentence, *before* the token is requested — never buried in terms. It is the user's decision to make, not ours to make for them.
- **SRC-9** — A user token grants everything that account can see, including DMs. The channel allowlist is therefore the entire privacy boundary and must be enforced at the reader, not by filtering downstream. `[HARD]`
- **SRC-10** — Discord tokens expire on password change, logout-everywhere, or a forced re-auth. Detect invalidation, tell the user in-channel, and prompt for a new one. **A dead reader must never fail silently** — the desk would go quiet on a source the user believes is live.
- **SRC-11** — History backfill on a Discord source is paced, not bulk-pulled. It is the single most detectable moment and the one most likely to cost the user their account.

### When a source can't be read

> **Silence is the failure mode that matters.** A user who believes a source is live while it is dead is missing calls and does not know it. Every access failure must surface quickly and name the source — a broken reader that says nothing is worse than one that never worked.

- **SRC-12** — Every source carries a health state — *live*, *degraded*, or *dead* — queryable by the user at any time in plain language.
- **SRC-13** — Transient failures reconnect silently. Gateway drops, brief rate limits and network blips never reach the user. **Escalate after 5 minutes of continuous failure, or three failed reconnects, whichever comes first.** `[HARD]`
- **SRC-14** — Permanent failures alert in-channel within minutes, naming **the specific source**, the most likely cause, and the single action that fixes it. Never a generic error.
- **SRC-15** — Failure causes are distinguished, because they need different messages: invalid or rotated token · account no longer in the server · channel no longer visible to that role · channel deleted · account suspended · rate limited.
- **SRC-16** — At configuration time, partial success is reported as such. If two channels connect and one does not, say exactly that rather than failing the whole setup.
- **SRC-17** — A dead source is marked degraded, never silently dropped. Its score freezes rather than decaying — it stopped producing because we lost access, not because it got worse.
- **SRC-18** — **Silence detection.** Flag a source for a health check at **three times its own median gap between messages, with a floor of four hours** — per source, never a global constant, since a busy call group and a slow research feed have nothing in common. A quiet group and a broken reader look identical from the outside, and the desk must not assume the first. `[HARD]`
- **SRC-19** — Losing a source never blocks the desk. The remaining sources continue, and the loss is reported as reduced coverage rather than an outage.

### X specifics

X is the one source type needing no user credential — public posts are public, so one platform-wide key serves everyone. Users pick handles; nothing can break when they change a password.

- **SRC-20** — X ingestion runs on a single platform-held key. User OAuth exists only to import their follows and Lists as a picker convenience — never as an access mechanism.
- **SRC-21** — Handle ingestion is shared across users. The union of all tracked handles is polled once; two hundred users following the same account is one stream, not two hundred. `[HARD]`
- **SRC-22** — Real-time X arrives by push, not polling. Sub-second detection on tracked accounts is a requirement, not an optimisation — a 30-second poll interval is 30 seconds of the only window that matters.
- **SRC-23** — **Screenshots must be read.** A large share of calls are images, not text. Without OCR on attached media those calls are invisible, and the desk will believe a source is quiet when it is not. `[HARD]`
- **SRC-24** — A cashtag alone never surfaces a candidate. One ticker maps to many tokens on a permissionless chain, so `$TICKER` may only corroborate a candidate already identified on-chain — never originate one. `[HARD]`
- **SRC-24a** — **Cashtag ambiguity resolves to nothing.** Match against candidates active in a recent window; count it only if exactly one matches. Two or more matches means it counts toward none — a wrong attribution corrupts the source score that everything else depends on, and under-counting is the cheaper error. `[HARD]`
- **SRC-24b** — **Structured links are parsed, not fetched.** DexScreener, GeckoTerminal, Blockscout, PONS, Pools.trade and hood.fun carry the contract in the URL path — regex extraction, no network call. `t.co` wrappers are expanded first and cached permanently by shortlink.
- **SRC-24c** — Unknown links resolve in a **background worker, never the hot path**. Fetch once, extract addresses by regex, cache the URL-to-contract mapping permanently. Latency is irrelevant because on-chain detection has usually found the token already.
- **SRC-24d** — The URL resolver is sandboxed with no internal network access, an aggressive timeout, a size cap, a single redirect hop and a domain denylist. **Page content is regex-scanned only and never reaches a model** — otherwise anyone can put instructions on a webpage and tweet the link. `[HARD]`
- **SRC-24e** — A token referred to only by name, with no ticker, address or link, is ignored. Unresolvable, and on-chain detection surfaces the token regardless.
- **SRC-25** — **Deletions are scored.** A deleted call counts against its source's record rather than vanishing from it. Callers who scrub their losers are exactly who the scoring exists to catch.
- **SRC-26** — Reply and quote velocity feed propagation-stage classification. On X, that is the cleanest available measure of how far something has already spread.
- **SRC-26a** — **Engagement is measured against the author's own baseline, never in absolute terms.** 200 reposts on an account that normally gets 15 is a signal; 200 on an account that normally gets 5,000 is a dud. Raw counts are purchasable; deviation is expensive to fake. `[HARD]`
- **SRC-26b** — Quotes weigh more than likes for propagation — likes are approval, quotes are spread. A high reply-to-like ratio indicates argument rather than enthusiasm and counts against, not for.
- **SRC-26c** — **Follower count carries zero weight.** It is bought, as is the verification badge and most engagement. The only unpurchasable author signal is their measured track record, which we compute ourselves. `[HARD]`
- **SRC-26d** — Paid promotion is detected by **template similarity against the author\'s own recent posts** — shill copy is structurally repetitive with the ticker swapped. Nearly free, and combined with outcome scoring it removes shillers within weeks.
- **SRC-26e** — **On-demand reach lookup.** When a user asks about a token no tracked source has mentioned, search X at query time rather than standing ingestion. About a cent per question, cached in Redis so repeats are free. The desk states whether it is reporting reach among trusted sources or a wider search — never implying it has seen everything.

### Chat specifics

Telegram and Discord offer almost none of X's signals, so the scoring model differs. The reference-resolution layer — addresses, links, cashtags — is shared and identical.

- **SRC-30** — The scoring unit is the **(channel, author) pair**, not the channel. A group may run 8% overall while one caller inside it runs 35%; scoring the channel alone discards that. `[HARD]`
- **SRC-31** — **First mention only.** One person posts the call and fifty reply. Only the first occurrence per (channel, token) counts — counting the echo would inflate every group with an active chat. `[HARD]`
- **SRC-32** — Propagation for chats is measured **across channels, not within one**. A chat message reaches everyone instantly with no retransmission signal, so intra-channel engagement says nothing about spread. One channel is early; twelve is spreading.
- **SRC-33** — Reactions carry little weight — trivially gamed in a small server. Use velocity against that channel's own baseline, or disregard.
- **SRC-34** — Deleted messages count against the caller's record rather than vanishing from it, exactly as on X.
- **SRC-35** — **Chat images are OCR'd locally.** Screenshot calls are as common in chats as on X, and no vendor covers Telegram or Discord. Without it the desk believes a source is quiet when it is posting. `[HARD]`

### Thread context

> **The thread that matters does not exist yet when you decide.** At t+35s a thread is three replies deep and says nothing; the person who digs into the deployer posts at t+10 minutes. **Thread analysis is therefore an abort mechanism, not an entry one** — it is the best available input to the desk changing its mind, and a poor input to the initial call.

- **THR-1** — Ingest stores a **reference, never the text** — conversation and post ID on X, channel and message ID on Telegram and Discord. Context is retrievable without being retained.
- **THR-2** — Thread context is fetched **only for candidates that already survived** scoring. Reading threads on the raw stream would cost thousands a month and buy nothing. `[HARD]`
- **THR-3** — Re-fetch on a schedule after the call — roughly t+90s and t+5m — because the useful analysis appears after the decision, not before.
- **THR-4** — The model **extracts structured claims; it never returns a verdict**. Claims made, concerns raised, whether the token is contested — validated against a schema. Scoring stays deterministic and the decision stays with the Chair. `[HARD]`
- **THR-5** — **Thread content is untrusted input.** Once the product is known, scammers will write messages engineered to steer it. Content is data, never instruction, and no extracted field may directly set a score. `[HARD]`
- **THR-6** — Extracted claims are weighted by the **source score of whoever made them**, not by consensus. Ten unknown accounts agreeing is worth less than one proven caller raising a doubt — otherwise a thread of shills reads as confirmation.
- **THR-7** — A concern raised by a high-scoring participant can trigger an **abort**, which is exempt from the tag budget like every other exit message.
- **THR-8** — Thread content from a user's **private source stays scoped to that user**. It may inform their decision and never enters cohort or global memory. `[HARD]`

### Per-user limits

- **SRC-27** — Soft limits, not hard rejections: **100 X handles · 25 Telegram groups · 10 Discord channels.** Past the limit the Chair says what it will cost in noise and asks, rather than refusing.
- **SRC-28** — Limits are generous on X because cost follows the union of handles, not the sum per user. They are tighter on Telegram and Discord because each carries a live session and real per-source overhead.
- **SRC-29** — Before proposing a limit increase, the Chair proposes a cull. A user at the cap almost always has sources below 10% that should go first.

## 06. Source scoring

Scoring sources, not just tokens, is the component that turns a firehose into a filter. It compounds and is the primary data moat.

- **SCR-1** — Every source carries a rolling record: hit rate, median return, time-to-peak, rug rate, measured at 1h / 6h / 24h after the call.
- **SCR-2** — On add, backfill. Pull the source's scrollable history, extract every contract address with timestamp, retrieve historical price and liquidity, and score retroactively. Target: usable score in minutes, not weeks. `[HARD]`
- **SCR-3** — New sources enter shadow mode: they contribute to discovery but carry zero weight toward the alert threshold until scored across ~20–30 calls or 14 days.
- **SCR-4** — Correlation clustering. Sources that habitually co-fire within a short window count as one source, not many. Confluence is measured over independent sources only.
- **SCR-5** — Source scores are shown to the user, with a prompt to drop poor performers.
- **SCR-6** — Wallet reputation table refreshed hourly. Deployer reputation table maintained continuously from chain history.

## 07. Pipeline and latency

> **Entry window.** Block 0 is not the target — it is unwinnable against sniper bots and is a coin flip on rugs and bundled supply. The target is minutes 1–5: after the bots have shown their hand, well before the crowd.

| t | Stage | Budget |
|---|---|---|
| 0s | Mint or pool creation detected — WebSocket subscription | <500ms |
| 0.5s | Kill filters — pure code, parallel, no network calls | <100ms |
| 2s | Enrichment — deployer, holders, LP state. Cache lookups only | <1.5s |
| 5–30s | Confirmation window — first buys, smart money, liquidity growth | 25s |
| ~35s | Score clears threshold, desk reviews, alert fires | — |
| 35s+ | Continuous monitoring, abort available | Ongoing |

### Funnel stages

Ingest → kill filters → cluster and dedupe by token → propagation-stage classification → score → suppression check → route to ambient or tag. Thousands per hour enter; a handful per day reach the user.

- **PIPE-1** — Event-driven throughout. No polling in the hot path. `[DETERMINISTIC]`
- **PIPE-2** — All reputation data precomputed and served from cache. An uncached RPC call in the hot path is a defect.
- **PIPE-3** — No LLM invocation before t+35s. Stages 1–4 are compiled or fast-path code. LLM cost occurs only at final review. `[HARD]`
- **PIPE-4** — Kill filters reject on: mint authority live, freeze authority live, LP unlocked or unburned, deployer linked to prior rugs, top-10 concentration above threshold, honeypot simulation failure, liquidity below floor, token already alerted.
- **PIPE-5** — The confirmation window is mandatory and cannot be skipped for speed. The first seconds after mint are bot activity and carry no signal.
- **PIPE-6** — Every mention is logged with source, token and timestamp, permanently. Source scoring cannot be reconstructed retroactively without it.
- **PIPE-7** — The propagation-stage classifier is the highest-value component. Everything else is plumbing.

## 08. Triggers and suppression

| Trigger | Definition |
|---|---|
| Velocity break | Mentions per minute crosses threshold, weighted by source score. Rate of change, not count |
| Confluence | N uncorrelated high-score sources hit the same address within a window |
| Tier-1 solo | A single source with exceptional hit rate speaks. Sufficient alone |
| Private source | A source the user personally added fires. Weighted higher; that user only |
| Propagation gap | Live on X but not yet in Telegram, or the reverse. Inside the diffusion lag |
| Narrative ignition | A theme spikes before any token attaches. Desk goes hunting |
| Smart-money cluster | Multiple tracked profitable wallets enter independently within a window |
| Organic growth curve | Fresh pool with healthy liquidity and holder growth, buy/sell ratio intact |
| Repeat deployer | An address with prior successful launches ships again |

### Suppression

- **TRG-1** — Mention count already high → classified late → suppressed. Loudness is a reason to stay silent. `[HARD]`
- **TRG-2** — Correlated sources collapse to one signal.
- **TRG-3** — A token is never alerted twice.
- **TRG-4** — Daily tag budget exhausted → desk stays quiet regardless of score.

## 09. Interaction model

- **INT-1** — All desk activity is visible in the channel and silent by default. Discussion never produces a notification.
- **INT-2** — The `@` tag is the only push notification in the system. Budget applies to tags, not messages — discussion is unlimited. `[HARD]`
- **INT-3** — Tag budget: 3–5 per day, enforced in code.
- **INT-4** — A tag requires all four: a specific decision, a concrete proposal with size and exit, a response window, and a defined default if the user does not reply.
- **INT-5** — The tag message is self-contained and readable without scrolling. Preceding discussion is context, not a prerequisite.
- **INT-6** — Timeout behaviour by mode. *Advisory*: skip, and say so. *Supervised*: execute at reduced size, or skip above threshold. *Discretionary*: execute as proposed.
- **INT-7** — Response windows: ~90s for Book C, minutes for B, hours for A.
- **INT-8** — One candidate is one thread. Discussion, tag, reply, execution, monitoring and outcome live in a single readable unit.
- **INT-9** — One re-ping maximum, only for time-critical decisions, only inside the response window.
- **INT-10** — The Chair closes every thread it opens, including when the outcome is a loss.
- **INT-11** — Scheduled reports: a short morning brief, and a Friday review with attribution.
- **INT-12** — The user may interject at any time. Their stated view enters the record as a principal directive the desk must engage with.

## 10. Ask the desk

- **ASK-1** — The user can ask about any token by contract address, ticker, link or description. No command syntax.
- **ASK-2** — Disambiguate before analysing. On ticker collision the Chair asks which contract, showing chain, age and liquidity.
- **ASK-3** — Answers stream. Specialists post as they finish. First useful content within ~5s, complete within 30s.
- **ASK-4** — Cached verdicts return instantly. If the desk already rejected this token, say so and why.
- **ASK-5** — Every answer states what it means for this user specifically — correlated exposure, mandate conflicts, whether it fits what they said they wanted.
- **ASK-6** — The desk must be able to decline, and to name behavioural patterns — repeated queries after a loss, chasing a token already late on its curve.
- **ASK-7** — If the user names where they saw it, offer to add and backfill that source.
- **ASK-8** — Queries are logged and rate-limited. Aggregate query volume is a signal; one user's private-source interest is never exposed to another.

### Screening — the user asks for tokens

A third mode. Alerts are desk-originated; *ask* analyses a token the user names; **screening is the user originating a search with their own criteria**, in plain language, with no query syntax.

- **SCR-Q1** — Criteria are stated conversationally and translated into a structured filter by the desk. *"anything from the last hour with decent liquidity"* · *"3+ smart wallets in and a clean deployer"* · *"what did you kill today"*.
- **SCR-Q2** — Screens query the stored candidate history, not a fresh crawl. Everything the pipeline has scored is already retained, so a screen is fast and costs almost nothing.
- **SCR-Q3** — Results rank by **our score, not by how well they match the filter**. Matching a criterion and being worth buying are different things, and the desk says which is which. `[HARD]`
- **SCR-Q4** — **Returning nothing is a valid and common answer.** A screen that always produces three tokens is a call group with extra steps. `[HARD]`
- **SCR-Q5** — Every result carries its verdicts — forensics, flow, propagation stage — never a bare list of tickers. A list without judgement is a block explorer.
- **SCR-Q6** — Criteria the desk cannot evaluate are named as such rather than guessed at. If we have no data on it, we say so.
- **SCR-Q7** — Vague criteria are pushed back on. *"Find me a 10x"* is not a criterion, and the Chair says so and asks what they actually want.
- **SCR-Q8** — **Rejection is queryable.** Users can ask what was killed and why, with counts by reason. This is the most convincing transparency the product has and it costs nothing to expose.
- **SCR-Q9** — Any screen can be saved as a **standing watch** that tags the user when something clears it. This is how a user builds their own strategy inside the product without us building a strategy builder.
- **SCR-Q10** — Standing watches obey the same suppression and tag budget as desk-originated alerts. A user cannot bypass the noise controls by phrasing an alert as a saved search. `[HARD]`
- **SCR-Q11** — Screens are rate-limited separately from single-token queries, being materially more expensive.

### The conversational loop

> **Alerts are a pipeline. Conversation is a genuine agent loop.** A user can ask anything trading-related, and answering well means deciding what to fetch, reading the result, and deciding whether to fetch more. This is the one place a model chooses what runs next — and it is walled off from everything that touches money.

- **LOOP-1** — The conversational surface runs a **bounded tool-calling loop** over a fixed registry of our own read-only functions — token safety, hook verdict, candidate query, source score, positions, chain state, call history, thread context.
- **LOOP-2** — **Hard step cap.** Six tool calls, then it answers with what it has. No unbounded loop on a metered inference budget. `[HARD]`
- **LOOP-3** — Every tool is **read-only**. Nothing in the registry can place an order, move funds, change a mandate or alter a setting — those stay behind explicit confirmation flows. `[HARD]`
- **LOOP-4** — Tools take the acting user\'s identity and are subject to the same row-level security. A tool physically cannot read another user\'s private-source data. `[HARD]`
- **LOOP-5** — Tool parameters are schema-validated and tool results are treated as **untrusted data, never instruction**. A token name or description containing injected text must not steer the loop. `[HARD]`
- **LOOP-6** — The loop never runs in the hot path and never participates in a trading decision. It answers questions; the desk makes calls. `[HARD]`
- **LOOP-7** — T2 for routine questions, T3 only when the question genuinely needs reasoning. Tier is chosen per query, not fixed.

## 11. Conversation standard

- **CNV-1** — Messages are terse. Specialists: one line. Chair escalation: three lines maximum. Desk chatter, not conversation. `[HARD]`
- **CNV-2** — No ticker formatting, no emoji, no template blocks, no countdown timers rendered as UI. Numbers sit inside sentences.
- **CNV-3** — Agents disagree in public. Dissent is posted, not resolved silently.
- **CNV-4** — The Chair states its confidence and what convinced it, and names the cost of any override the user requests.
- **CNV-5** — Losses are reported in the same tone as wins, with the source's updated hit rate attached.

Example — escalation, Book C

Example — user query, refusal

## 12. Wallet and custody

The desk needs signing authority to trade. How it gets that authority determines both the security surface and the regulatory tier.

> **Custody changes the legal category.** Holding a user's private key makes us a custodian, regardless of what the product is called. Delegated and scoped-permission wallets deliver the same execution capability without that. Key import is offered because users ask for it, but it is never the default and never the recommended path.

| Mode | Mechanism | Who holds the key | Use |
|---|---|---|---|
| Generated burner | Desk creates a wallet at onboarding, key lives in the signing service | Us | Default for Book C. Funded deliberately, capped |
| Delegated agent wallet | Venue-native trade-only delegation, cannot withdraw externally | User | Preferred wherever the venue supports it |
| Smart account | Safe with scoped module, or ERC-4337 session keys with asset and venue allowlists | User | Preferred on EVM |
| MPC | Key split across parties, no single holder | Shared | Where we must sign but should not hold |
| Imported key | User supplies a private key | Us | Opt-in only, with explicit warning |

- **WAL-1** — Onboarding offers wallet generation first. Import is a secondary path behind an explicit consent screen stating that we will hold the key.
- **WAL-2** — Where a venue supports trade-only delegation or scoped session keys, that path is default and import is disabled for it.
- **WAL-3** — Private keys never touch the application tier. Signing runs in an isolated service; the app sends unsigned intents and receives signed transactions. `[HARD]`
- **WAL-4** — Keys encrypted at rest with per-user data keys under a KMS or HSM root. Never logged, never in error traces, never in backups in plaintext. `[HARD]`
- **WAL-5** — Generated wallets are exportable by the user at any time. We do not hold anyone hostage to the product.
- **WAL-6** — The trading wallet is separate from the user's main holdings. Only funds deliberately transferred into it are ever at risk. Onboarding says this plainly.
- **WAL-7** — Multi-chain: one wallet set per supported chain, shown as a single balance view.
- **WAL-8** — Withdrawal to an external address is a user-only action. No agent may initiate an outbound transfer to an address outside the user's own set. `[HARD]`
- **WAL-9** — Key rotation and wallet retirement supported without losing position history.

## 13. Execution agent

Execution is a separate member of the same channel. It has no opinions and originates nothing — it receives a signed order from the Chair and reports what happened.

- **EXE-1** — Execution accepts orders from the Chair only. It never originates a trade, never resizes, never overrides. `[HARD]`
- **EXE-2** — The order object is explicit and complete: token, chain, venue, side, size, max slippage, priority fee ceiling, TTL, and the exit plan to arm on fill.
- **EXE-3** — Risk performs a pre-trade check on every order — concentration, mandate conflicts, wallet balance, gas reserve. Deterministic, and it can reject. `[DETERMINISTIC]`
- **EXE-4** — Two entry paths, per the autonomy mode. *Auto*: order arrives pre-authorised under the user's settings and executes immediately. *Consent*: the Chair tags the user, the user approves, the Chair hands the order to Execution.
- **EXE-5** — Every transaction is simulated before broadcast. A failed simulation aborts and posts the reason. `[HARD]`
- **EXE-6** — Orders carry a TTL. An order older than its TTL is cancelled, not executed at a stale price. Book C default 20s.
- **EXE-7** — Idempotency keys on every order. A retry can never produce a second buy. `[HARD]`
- **EXE-8** — Gas reserve is untouchable. The wallet's native balance never falls below the level needed to exit every open position.
- **EXE-9** — Private mempool or MEV-protected submission by default where available.
- **EXE-10** — Nonce management, stuck-transaction detection, replacement and cancellation handled without user involvement.
- **EXE-11** — Token approvals are exact-amount, never unlimited. Stale approvals revoked on position close.
- **EXE-12** — Circuit breaker: N consecutive failures, or a daily loss limit, halts all execution and notifies the user.
- **EXE-13** — Global kill switch stops execution instantly across all users and all chains, reachable by the operator and by the user for their own wallet. `[HARD]`
- **EXE-14** — Execution posts a fill report to the thread: actual price, actual slippage against expected, gas paid, venue, tx link. Failures post the reason in the same place.
- **EXE-15** — Exit orders take priority over entry orders in the queue at all times.

Example — handoff on consent

## 14. Trading settings

Set once at onboarding, editable any time in conversation. Every setting is scoped per book, because Book A and Book C need opposite values.

| Setting | Scope | Typical | Notes |
|---|---|---|---|
| Buy size | Per book | % of book or fixed | Book C fixed and small; A and B percentage-based |
| Max slippage | Per book | A 0.5% · B 2% · C 10–15% | Exits allow a higher ceiling than entries |
| Gas cap | Per book | Absolute ceiling | Robinhood Chain orders by arrival, not by fee — paying more buys nothing, so this is a cost limit rather than a bidding lever |
| Gas reserve | Per wallet | Cover all open exits | Never spendable |
| Auto-execute ceiling | Per book | e.g. ≤1% of portfolio | Above it, the Chair must tag for consent |
| Max open positions | Per book | User-set | Prevents dilution into unmonitorable breadth |
| Order TTL | Per book | C 20s · B 5m · A 1h | Stale orders cancel, never execute |
| MEV protection | Per chain | On | Private submission where supported |
| Auto-sell | Per position | On | Ladder and invalidation always armed on fill |
| Cooldown | Per book | User-set | Minimum interval between entries after a loss |

- **SET-1** — Settings are captured during onboarding with sane defaults per book. The user can accept defaults and start.
- **SET-2** — All settings editable in conversation, in plain language, without a settings screen.
- **SET-3** — Auto-execution requires an explicit ceiling. There is no unlimited auto-trade mode. `[HARD]`
- **SET-4** — Settings that widen risk — larger size, higher slippage, higher auto-execute ceiling — require confirmation and are logged with a timestamp.
- **SET-5** — Setting changes never apply retroactively to armed exits on open positions.
- **SET-6** — A user setting can never exceed a mandate limit. The mandate is the ceiling; settings operate beneath it. `[DETERMINISTIC]`
- **SET-7** — Fees are disclosed at the point of every fill — gas, venue fee, and our fee, itemised.
- **SET-8** — Auto-execution pauses automatically on drawdown breach and resumes only on explicit user action.

## 15. Portfolio and reporting

Everything readable in the channel, on request or on schedule. No dashboard required to operate the product.

- **PNL-1** — Holdings on request: token, size, entry, current, unrealised P&L, share of book, age, thesis status.
- **PNL-2** — Trade history: fills, timestamps, venue, actual slippage, gas, realised P&L, and the call that originated it.
- **PNL-3** — P&L is net of everything — gas, venue fees, our fee. Gross figures are never shown alone. `[HARD]`
- **PNL-4** — Cost basis accounted FIFO per token per wallet, surviving partial fills and ladder exits.
- **PNL-5** — Realised and unrealised reported separately, day, week, month and since inception.
- **PNL-6** — Scheduled: a short daily position and P&L line, and the Friday review with attribution by agent and by source.
- **PNL-7** — Every closed position reports outcome against its original thesis — did the invalidation fire, did the ladder run, did the time stop trigger.
- **PNL-9** — Full export on request — CSV of fills and positions, for tax and for the user's own records.
- **PNL-10** — Reporting is terse per CNV-1. Long tables go to an attached export, not into the channel.

Example — holdings on request

## 16. Exit engine

The exit is the half that makes recommending fresh mints defensible. Book C tokens can die within hours.

- **EXIT-1** — No call ships without an exit plan. Entry-only recommendations are prohibited. `[HARD]`
- **EXIT-2** — Ladder: pre-set scale-outs defined at entry — recover cost, trim, trail the remainder.
- **EXIT-3** — Hard invalidation: liquidity pulled below floor, dev wallet movement, mint authority reappears, top-10 holder sells above threshold.
- **EXIT-4** — Soft decay: volume decay rate, buy/sell ratio flip, holder growth stall, social velocity rolling over. Exit on momentum death, not price alone.
- **EXIT-5** — Time stop. No thesis progress within the book's window means out.
- **EXIT-6** — Exit monitoring runs per open position from the moment of entry, on the reflex path with no LLM in the loop. `[DETERMINISTIC]`
- **EXIT-7** — Exit and abort notifications are exempt from the tag budget. Safety messages always reach the user. `[HARD]`

## 17. Access and payments

One price, one month of access, paid from the user's own wallet on Robinhood Chain. There is no card processing, no merchant of record, no tax registration and no web checkout beyond a single wallet page.

> **Telegram constraint.** A bot cannot create a group. Provisioning a private channel per user requires an MTProto client account to create the supergroup, add the agent bots, promote them, and issue an invite. Account-level rate limits apply, so provisioning runs off a pool of concierge accounts with a queue behind it. This is the single hardest part of onboarding to scale — design it before launch, not after.

### The offer

```
  One month of access — $99

  [ Pay for a month ]   ends after a month. Buy another if you want.
  [ Pay monthly ]       auto-renews from your wallet. Cancel any time.
```

- **PAY-1** — One price, one plan, one month of access. No tiers, no plan matrix, no upgrade paths. `[HARD]`
- **PAY-2** — Two options at the same price: pay for a month, or auto-renew from the wallet. Both resolve to the same subscription object — nothing downstream branches on which was chosen.
- **PAY-3** — Payment is a stablecoin allowance signature on Robinhood Chain, pulled by us. The user signs, never sends — no gas, no transaction, no address to copy, no amount to type.
- **PAY-4** — The one-month option must be structurally incapable of being charged twice, not merely policed by our code. `[HARD]`
- **PAY-5** — Auto-renew is charged monthly until cancelled. Cancellation by one message to the Chair, and revocation from the wallet, are both honoured immediately and equally.
- **PAY-6** — Allowance amounts never appear in product copy. The wallet displays what it is signing; that is its job, not ours.
- **PAY-7** — Access is always stated as a date, never a duration. *"You're in until 7 October"*, not *"one month remaining"*.
- **PAY-8** — Reminder three days before expiry, in-channel, with one button to buy another month.
- **PAY-9** — A failed pull — insufficient balance or revoked allowance — is stated plainly in-channel, followed by grace, then deprovision. No aggressive retries, no silent failure.
- **PAY-10** — Sanctions screening on the paying address before access is granted. Non-negotiable when receiving funds directly. `[HARD]`
- **PAY-11** — Jurisdiction gates which books a user may enable — stock tokens are unavailable to US persons.
- **PAY-12** — Subscription funds and trading wallets are entirely separate systems. Subscription money never touches a trading wallet and vice versa. `[HARD]`
- **PAY-13** — Per-trade and performance fees are out of scope for v1. They change the regulatory category and are a separate decision.

### Provisioning

- **PRV-0** — **Identity is bound before payment, not after.** The user starts with the Chair bot, so we hold their Telegram ID before any wallet step. A signup that drops mid-payment is still reachable — the Chair follows up. This is the whole reason the flow runs Telegram → wallet → Telegram. `[HARD]`
- **PRV-1** — The Chair issues a single-use setup link binding that Telegram user to one payment session. Tokens expire, cannot be reused, and the resulting wallet address is bound to that user permanently. `[HARD]`
- **PRV-2** — A concierge MTProto account creates the private supergroup, adds the agent bots, promotes them, sets permissions, and invites the bound user.
- **PRV-3** — Provisioning is queued and runs off a pool of concierge accounts with per-account rate limits respected. Failures retry; the user sees a status, never a dead end.
- **PRV-4** — Any account joining a user channel that is not the bound user or an agent bot is removed automatically. `[DETERMINISTIC]`
- **PRV-5** — Invite links are single-use and revoked immediately after the bound user joins.
- **PRV-6** — Onboarding begins in-channel the moment the user joins — mandate conversation first, then wallet, then trading settings, then sources.
- **PRV-7** — Time from payment to a Chair greeting in the channel: under 60 seconds at p95.
- **PRV-8** — On churn, the user is removed from the channel and the channel is archived, with the mandate, history and P&L retained and restorable on resubscribe.
- **PRV-9** — Deprovisioning revokes source sessions and halts execution before removing access. Money stops before the chat does. `[HARD]`
- **PRV-10** — A user's funds remain theirs after churn. Wallet export is available regardless of subscription state.

### Entitlements

- **ENT-1** — Access is binary: active or expired. There are no tiers to gate, so there is no entitlement matrix to build. `[HARD]`
- **ENT-2** — Autonomy mode is a user setting under their mandate, not something they pay more for.
- **ENT-3** — Access checks are enforced server-side at the point of action, never by hiding a control.
- **ENT-4** — Expiry reverts the user to read-only in-channel before removal. **It never abandons an open position or disarms an exit.** Someone who stops paying still gets told to sell. `[HARD]`
- **ENT-5** — LLM and data cost per user is tracked against the single price continuously, with an alert when a cohort's margin inverts.

## 18. Accountability

- **ACC-1** — Every position carries a written thesis with an explicit invalidation condition. Agents monitor for that condition continuously.
- **ACC-2** — Dissent is recorded when overruled, and scored against the outcome later.
- **ACC-3** — Monthly attribution by agent. Which agent's conviction drove which P&L.
- **ACC-4** — The user can bench, reweight or add agents based on their record.
- **ACC-5** — Every alert is logged with full system state at fire time, so our own hit rate is measurable and publishable.
- **ACC-6** — Track record is published unedited before any subscription is sold — wins and losses.

## 19. Security and privacy

- **SEC-1** — Telegram session strings **and Discord account tokens** encrypted at rest with per-user keys, in a store isolated from all other data, never logged, never in error traces or plaintext backups. Together they are the highest-value target in the system — a breach costs users their accounts, not just their data. `[HARD]`
- **SEC-2** — Allowlist enforced at the reader client, not by server-side filtering.
- **SEC-3** — Store extracted signal — address, timestamp, source ID. Drop raw message text after extraction.
- **SEC-4** — Exchange API keys accepted trade-only, withdrawals disabled. Reject any key with withdrawal permission. `[HARD]`
- **SEC-5** — On-chain execution via delegated agent wallets or session keys that cannot withdraw to an external address. We never take custody.
- **SEC-6** — Per-source revoke and a global kill switch, reachable in one action.
- **SEC-7** — Private-source isolation enforced at the data-model level, with a test that fails the build if signal crosses users.

## 20. Scalability

> **The rule the whole system scales on.** Run the pipeline once globally, personalise only the last step. Detection, kill filters, enrichment, clustering, propagation classification and scoring are identical for every user and happen once per token. Only sizing, mandate check and message rendering are per user. Do this and a new user costs almost nothing; skip it and inference cost kills the product at roughly fifty users.

- **SCA-1** — Shared analysis, personal decision. Candidate evaluation is computed once and fanned out. Per-user work is bounded to sizing, mandate evaluation and rendering. `[HARD]`
- **SCA-2** — The only per-user analysis is evaluation of that user's private sources. Cost per user scales with their private source count, not with total system activity.
- **SCA-3** — Ingest workers partition by chain and by source. Decision workers partition by user. All workers stateless and horizontally scalable.
- **SCA-4** — Per-group outbound message queue respecting Telegram's ~20 messages per minute per group. Exits and aborts jump the queue; ambient chatter is dropped under pressure, never delayed indefinitely. `[DETERMINISTIC]`
- **SCA-5** — Concierge account pool for provisioning, with per-account daily creation limits, health monitoring, automatic rotation and a queue. Provisioning rate is paced to signup rate, never bursted.
- **SCA-6** — Backpressure sheds work in a defined order: ambient discussion first, then low-score candidates, then Book B, then Book A. Exit monitoring and kill switches are never shed. `[HARD]`
- **SCA-7** — Target: 1,000 concurrent user channels on the shared pipeline without architectural change. Beyond that, Telegram stops being the primary surface.
- **SCA-8** — Source ingestion cost is shared. Two users subscribed to the same group are served by one reader, one extraction and one score.

## 21. Reliability

One question defines the reliability posture: if every LLM provider is down, does the system still get people out of positions? It must.

- **REL-1** — Exit monitoring, stop execution and kill switches run entirely on deterministic code with no model dependency. They survive a total LLM outage. `[HARD]`
- **REL-2** — Degradation ladder: full desk → no discussion, calls still issued → deterministic only, exits and stops only → halt entries, exits continue. Each level is explicit and announced in-channel.
- **REL-3** — Every external dependency has a declared fallback: two RPC providers per chain, two data providers per source type, two model providers per tier.
- **REL-4** — At-least-once delivery everywhere, with idempotency keys making duplicates harmless. Never at-most-once on anything touching money. `[HARD]`
- **REL-5** — Silence detection on every listener. A feed that stops delivering is more dangerous than one that errors, and must alert within 60s.
- **REL-6** — Full system state reconstructable by replaying the event log. Positions and theses are derived, not primary.
- **REL-7** — Dead-letter queues with replay for every async path.
- **REL-8** — Scheduled failure drills in staging: kill the model provider, kill the primary RPC, kill the data feed. Exits must still fire in all three.
- **REL-9** — SLOs: 99.9% availability on exit monitoring and execution; 99% on desk discussion. The two are deliberately different.
- **REL-10** — Continuous operation. There is no market close and no maintenance window that stops exit monitoring.
- **REL-11** — Latency: mint to tag ≤60s p95 for Book C; kill filters and enrichment ≤2s from detection; exit triggers ≤1s from the on-chain event.

## 22. Cost and model routing

Most of this system is not an AI problem. The economics depend on keeping it that way.

| Tier | Work | What runs it | Share of decisions |
|---|---|---|---|
| T0 | Kill filters, scoring, risk checks, sizing, exit triggers | Plain code. No model | ~95% |
| T1 | Address extraction, spam and bot classification, dedupe, routing, cache lookups | Cheapest small model class, roughly $0.03–0.25 per M input | ~4% |
| T2 | Narrative read, forensics summary, message drafting, query answering | Mid tier, roughly $1/$5 per M | ~1% |
| T3 | Chair synthesis on a real decision, weekly review, mandate conversation | Frontier, any provider | <0.1% |

- **COST-1** — Every task is assigned the lowest tier that can do it. Promoting a task to a higher tier requires evidence that the lower one failed. `[HARD]`
- **COST-2** — No model call in the hot path before t+35s. T0 only. `[HARD]`
- **COST-3** — All model access goes through a provider-agnostic gateway. Models are configuration, never code, and are mixed freely across vendors by task.
- **COST-4** — Per-tier failover to a second vendor. A single provider outage degrades quality, never availability.
- **COST-5** — Semantic and exact-match caching at the gateway. Shared analysis is computed once and reused across every user who sees that candidate.
- **COST-6** — Prompt caching enabled on every repeated system context. Agent role definitions are static and must never be re-billed as fresh input.
- **COST-7** — Cost attributed per user, per agent and per task type. A cohort whose margin inverts triggers an alert, not a surprise at month end.
- **COST-8** — Hard monthly inference ceiling per user, enforced by circuit breaker. Breaching it degrades to lower tiers rather than overspending. `[DETERMINISTIC]`
- **COST-9** — Non-urgent work — backfills, scoring, reviews, attribution — runs on batch pricing where the provider offers it.
- **COST-10** — Model choice is benchmarked per task against outcome quality, not vendor preference. Re-evaluated quarterly; the cheap tier moves fast.

## 23. Memory and knowledge

The desk is exposed to an enormous volume of chat and social text. Almost none of it belongs in a model's context.

### Three scopes

| Scope | Contents | Shared with |
|---|---|---|
| Global | Deployer and wallet reputation, base source scores, token outcome history, market regime, contract risk patterns | Everyone |
| Cohort | Extracted signal and scores for a public or base source | Every user subscribed to that source |
| User | Mandate, positions, trade history, overrides, behavioural profile, conversation history, private-source signal | That user only |

- **MEM-1** — Scopes never mix. Anything derived from a private source is user-scoped permanently and can never be promoted to cohort or global. `[HARD]`
- **MEM-2** — Raw text is not memory. Messages and posts are parsed to structured facts — address, source, timestamp, claim type, sentiment — and the raw text is discarded. `[HARD]`
- **MEM-3** — Four layers, each smaller than the last: raw ingest (transient, seconds) → extracted facts (structured, permanent) → rolling summaries (per source, per token, per period) → durable profile (user and source, slow-changing).
- **MEM-4** — Hard token budget per agent invocation. Retrieval is ranked and truncated to fit; context is never allowed to grow with activity. `[DETERMINISTIC]`
- **MEM-5** — Retrieve, never stuff. Agents receive the top-k relevant facts for the decision at hand, not a window of recent chatter.
- **MEM-6** — Time decay by class. Market context expires in hours, source scores decay over months, user profile and trade history do not expire.
- **MEM-7** — Contradiction handling is temporal. Newer facts supersede older ones and the supersession is recorded, so the desk can say what changed and when.
- **MEM-8** — The user profile captures how they talk, how they size, what they override, and how they behave after a loss. It informs tone and refusals, never the analysis itself.
- **MEM-9** — Every fact carries provenance — which source, when, verified on-chain or not. Unattributed facts are not admissible into a decision.
- **MEM-10** — Deletion is real. A user's removal purges user-scoped memory; global and cohort knowledge derived from public sources persists.

## 24. Buy versus build

Default to buying. Three things are ours because they are the product: the propagation-stage classifier, source scoring, and desk orchestration. Everything else is a vendor or a library.

| Need | Use | Note |
|---|---|---|
| X / Twitter data | twitterapi.io, Bright Data, Apify; WebSocket relays for real-time | Roughly $0.15–0.70 per 1k tweets against $5 per 1k on the official API. Full firehose is enterprise-priced and unnecessary — we track curated accounts |
| Chain streaming | Forked Chainstack sequencer-feed decoder + Alchemy WebSocket | Pre-execution visibility from the Nitro relay. Do not write a decoder |
| Token safety | GoPlus Token Security API | Free, live on 4663. Honeypot, mint/blacklist, hidden fees, holder distribution. Do not reimplement |
| Telegram ingest | Telethon or TDLib, self-hosted | **No vendor exists.** The only component we are forced to build and operate ourselves. Plan account pooling and health monitoring accordingly |
| Discord ingest | Official Bot API | Admin-invited servers only. No compliant alternative |
| Swap routing | 1inch and 0x, quoted in parallel | Never write a router. Both live on 4663 from launch day |
| Hook analysis | HookScan, Hacken harness, Uniswap hook whitelist | Open source. Hooks are this chain's honeypot vector; skip pools whose hook is unlisted |
| Wallets and signing | Turnkey, Privy or Dfns | Hardware-isolated key management with a policy engine, from roughly $0.05 per signature. Building a signing service in-house is the worst possible use of engineering time |
| Model gateway | LiteLLM self-hosted, or OpenRouter managed | One interface, every vendor, per-task routing and failover |
| Agent memory | Mem0 to start; Zep where temporal reasoning over changing facts matters | Both open source. Do not build a memory layer |
| Vector search | pgvector on the primary Postgres | No separate vector database until volume forces it |
| Queue and cache | Redis Streams or NATS, Redis for hot state |  |
| Payments | Permit2 allowance, pulled by us | Canonical singleton, nothing to deploy. No cards, no merchant of record, no tax registration |
| Observability | Langfuse or Portkey for model traces and spend; standard APM for the rest |  |
| Existing bot code | Open-source honeypot and sniper implementations | Buy-then-sell fork simulation is a solved technique. Read the approach and port it rather than rediscovering it |

- **BUY-1** — Anything requiring dedicated infrastructure to operate is bought, not built, unless it is one of the three components named above.
- **BUY-2** — Every vendor is accessed behind our own interface, so it can be swapped without touching business logic. `[HARD]`
- **BUY-3** — Every critical vendor has an identified replacement and a tested migration path before it enters production.
- **BUY-4** — Vendor cost is tracked per unit of business value — cost per candidate evaluated, per call issued, per user per month.
- **BUY-5** — Prefer open source we can self-host once volume justifies it, over closed services that cannot be brought in-house.

## 25. Build sequence

- **P1 · Source scoring, backfilled** — Join the base groups, curate the X list, backfill history, score every source retroactively. No product on top. Output: the 15 sources out of 200 that are worth listening to.
- **P2 · Book C pipeline, one chain** — Firehose, kill filters, clustering, propagation classifier, scoring, suppression. Ends at a scored candidate, not a message.
- **P3 · Exit engine** — Built before the alerting layer. Deterministic monitoring, ladder, invalidation, decay, time stop. Nothing goes out that this cannot manage.
- **P4 · Execution layer** — Signing service, wallet generation, order object, pre-trade checks, simulation, idempotency, fills and position accounting. Run against own capital on a burner wallet before any user touches it.
- **P5 · The channel** — Roster, ambient discussion, Chair tagging, thread lifecycle, ask-the-desk, holdings and P&L. Advisory and consent modes. Public track record.
- **P6 · Onboarding and user sources** — Mandate conversation, wallet setup, trading settings, X OAuth, Telegram delegation, probation, isolation. Then auto-execution and the autonomy dial.
- **P7 · Payments and provisioning** — Checkout, entitlements, the concierge account pool and provisioning queue, deep-link binding, dunning and deprovisioning. Last, because it only matters once the calls are worth paying for.

## 26. Open risks

| Risk | Position |
|---|---|
| Regulatory tier | Advisory and user-executed trading sit well below discretionary management. Holding keys makes us a custodian; taking a performance fee changes the category again. Decide both before pricing |
| Key custody | Imported keys are the single largest liability in the system. Default to generation and delegation, isolate signing, and treat a key breach as company-ending in planning |
| Execution failure modes | Double-fills, stuck nonces, gas exhaustion and stale-price fills destroy money faster than bad calls. Idempotency, simulation and the gas reserve are not optional |
| Agent groupthink | LLMs converge and manufacture consensus. Separate contexts, adversarial prompting, and scoring that rewards being right and contrarian. Unsolved until measured |
| Latency vs deliberation | A committee takes 30s+. Resolved by alerting on deterministic score and letting the desk abort afterwards. Verify the abort path actually saves money |
| Unit economics | Continuous multi-agent inference against live markets is real spend. Model per-user cost before build |
| Discord coverage | Structurally worse than Telegram. Disclose it; do not engineer around ToS |
| Source redistribution | Leaking a user's paid group signal to other users ends the company. Enforce in the data model |
| Provisioning scale | Group creation needs MTProto accounts with real rate limits. A pool, a queue and account-health monitoring are required before any launch spike |
| Jurisdiction | This cannot be sold everywhere. Geo-restriction and sanctions screening sit at checkout, and the blocked list is a product decision, not a legal afterthought |
| Call quality | If the calls are not good, none of the above matters. P1–P5 exist to find that out on own capital before selling anything |

---

*Tragentify · tragentify.ai · Requirements v0.3 · 11 Sep 2026*
