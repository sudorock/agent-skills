# Terse

Structured, information-dense output with coordinate addressing. Same information, fewer tokens, every line referenceable.

## Install

```
npx skills add sudorock/agent-skills --skill terse
```

## Before and After

The same investigation, two formats.

### Standard Output (218 tokens)

```
## Gateway Latency Investigation

I've investigated the gateway latency issue you reported. Here's what I found:

The v1 gateway is currently experiencing a p99 latency of 340ms, which has been
exceeding our 200ms budget since April 12th. After looking into this, I believe
the root cause is connection pool exhaustion that occurs under peak load.

Here are the specific findings:

1. **Connection Pool Sizing**: The connection pool is currently configured for
   100 connections, but during peak traffic, demand reaches approximately 250
   connections.

2. **Connection Leak**: When connections fail, they are not being properly
   released back to the pool. This compounds over time as the available pool
   gradually shrinks.

Based on my investigation, I recommend the following:

- Add a `finally` block to ensure connections are always released, even on error
- Increase the pool size from 100 to 300 connections

These changes should bring the p99 latency back within our 200ms budget.
```

### Terse Output (55 tokens)

```
# 1 Gateway Latency
v1 gateway p99 340ms; exceeds 200ms budget since 04-12; cause: pool exhaustion

## 1.1 Findings
1. pool sized for 100 conns; peak demand ~250
2. failed conns not released; leak compounds over hours

## 1.2 Fix
1. + finally block; ensures conn release on error
2. ~ pool size 100 -> 300
```

### Token Reduction

| | Standard | Terse | Reduction |
|---|---|---|---|
| Tokens | 218 | 55 | **75%** |

No information lost. Both outputs convey the same findings, root cause, and fix. Terse eliminates preamble ("I've investigated..."), hedge phrases ("I believe"), filler ("Here are the specific findings"), and restated context. What remains is the signal.

Token savings compound over a conversation. A 20-message investigation at 75% reduction saves thousands of tokens, keeping more of the context window available for actual work.

## Coordinate Addressing

Every piece of content gets a stable address. Messages, sections, lines, and table cells are all referenceable.

```
# 3 Migration Plan              <- message 3
## 3.1 Breaking Changes          <- message 3, section 1
1. + OAuth2 PKCE replaces keys   <- @3.1.1
2. - X-Api-Key removed           <- @3.1.2
```

### Reference Instead of Restate

Later messages reference earlier content by coordinate instead of repeating it:

```
# 4 Pool Fix [supersedes @2]
1. cause: connection leak per @2.2 (unreleased-conns)
2. rollback via @3.3 (canary-rollout) if regression
```

`@2.2` points to message 2, line 2. The parenthetical gloss "(unreleased-conns)" gives enough context to follow without scrolling back. This avoids the common LLM pattern of restating prior findings in every response.

### Expand on Demand

Request more depth on any coordinate:

```
user: expand @3.3
```

```
# 5 @3.3 Rollout Strategy
phased canary -> percentage ramp; automated promotion gates

## 5.1 Canary Phase
1. deploy to 2 canary nodes (us-east-1a, eu-west-1a)
2. soak 30min; monitor error rate + latency + saturation
3. IF error rate > 0.1% OR p99 > 200ms THEN auto-rollback

## 5.2 Percentage Ramp
1. 10% -> 25% -> 50% -> 100%; 1h soak per stage
2. promotion gated on same thresholds as @5.1 (canary-gates)
3. rollback at any stage reverts to v1; no manual intervention
```

### Table Cells

Tables are addressable down to individual cells:

```
## 3.2 Client Tiers
1.
| # | tier     | deadline   | status |
|---|----------|------------|--------|
| 1 | internal | 2026-04-20 | [done] |
| 2 | partner  | 2026-04-25 | [wip]  |
| 3 | public   | 2026-05-01 | [open] |
```

`@3.2.1[2,3]` references row 2, column 3 of the table at line 1 of section 3.2 (the partner deadline: 2026-04-25).

## Symbolic Notation

Compact operators replace verbose phrases:

| Instead of | Write |
|---|---|
| "changed from X to Y" | `X -> Y` |
| "added a circuit breaker" | `+ circuit breaker` |
| "removed the old header" | `- X-Api-Key` |
| "modified the pool size" | `~ pool 100 -> 300` |
| "I think this is likely" | `[likely]` |
| "this is done" | `[done]` |
| "if X then Y otherwise Z" | `IF X THEN Y ELSE Z` |
