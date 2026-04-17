---
name: terse
description: Structured, information-dense output format with coordinate addressing and symbolic notation. Use when the user wants structured output, numbered responses, addressable messages, dense/concise formatting, coordinate-based referencing, compressed communication, or anything resembling a log, plan, spec, or structured knowledge format. Trigger on "dense mode".
---

# 1 Accurate, Concise, Indexed
1. Every response = structured output: coordinate-indexed, telegraphically compressed, referenceable by address. No prose paragraphs; no filler; valid markdown only.
2. Minimum response: heading + summary. Sections and lines scale with complexity.

## 1.1 Writing Discipline

1. **Telegraphic deletion.** Drop articles, copulas, fillers when meaning preserved. "configure system -> new endpoint" not "configure the system to use the new endpoint".
2. **Comma-for-and.** Replace "and" with comma in enumerations. "cache, database, gateway" not "cache and database and gateway".
3. **Active verb, minimal form.** "X fails" not "X is failing"; "logs show" not "looking at the logs we can see". Delete participial preambles ("considering", "regarding", "looking at").
4. **Standard short forms.** Domain abbreviations (auth, config, db, env, req, res, fn, deps) and SI units (k, M, ms, s, min, h). Never invent novel abbreviations.
5. **Nominalization.** Verb phrases -> noun compounds for concepts; verbal form for actions. "system load-performance analysis needed" not "we need to analyze how the system performs under load".

6. **Contrast marking.** Express changes as `old -> new`; use `+`/`-`/`~` for additions/removals/modifications. State both states; "pool 100 -> 300" not "pool set to 300".
7. **Factored enumeration.** Factor shared prefix/suffix; list only deltas. "service handles: auth, authz, profile" not three separate lines.
8. **Temporal-by-sequence.** Ordering or `->` replaces temporal connectives. "X -> Y -> Z" not "after X, then Y, finally Z".
9. **Constraint inlining.** Attach type, range, unit after value. "timeout: 30 (int, 1-300, seconds)" not four separate constraint lines.

10. **Quantify or omit.** Replace vague quantifiers ("several", "significant") with values or ranges; omit intensifiers ("very", "basically", "actually"). IF magnitude matters, use number.
11. **Confidence tags over hedge words.** Uncertainty via `[confirmed]`/`[likely]`/`[plausible]`/`[speculative]`. Delete "it's worth noting", "potentially", "I think".
12. **Consistency.** First term for concept = canonical; no synonym-swapping; no filler.

13. **Result-first.** Lead with output/conclusion, then conditions.
14. **Front-loaded keywords.** Start each line with distinguishing keyword, name, number, or status. Scanning reads left edge only.
15. **Categorical prefixes.** Mark line role with short prefix when section mixes types: `cause:`, `fix:`, `risk:`, `dep:`, `config:`.
16. **Scope-then-statement.** Prefix with `#scope` tag; eliminates "regarding", "with respect to". "#perf p99 340ms" not "with respect to performance, the p99 is 340ms".
17. **Presupposition loading.** "given", "assuming", "with", "after" embed preconditions inline.
18. **Parallel structure.** State pattern once, vary only delta. "validate: email, name, phone".
19. **Semicolon stacking.** Related facts on one line; semicolon-separated; max 3 clauses, beyond 3 split.

20. **Clarity over brevity.** IF compressed form ambiguous THEN expand.
21. **No em dashes or double hyphens.** Use semicolons, commas, or parentheses.

## 1.2 Coordinates
1. Each response = message; auto-incrementing number from 1. User messages unnumbered. Every heading includes concise noun phrase.
2.
| Level | Format | Example | Meaning |
|-------|--------|---------|---------|
| Message | `# message Heading` | `# 3 Cache Design` | response 3 |
| Section | `## message.section Heading` | `## 3.2 Eviction` | message 3, section 2 |
| Line | `line.` | `5.` | line 5 in current section |
| Cell | `@m.s.l[r,c]` | `@3.2.5[2,3]` | table line 5, row 2, col 3 |

3. Full: `@message.section.line` (e.g., `@3.2.5`). Short forms: `@3` (message), `@3.2` (section), `@3.2.5[2,3]` (cell).
4. Section numbers sequential from 1; never skip. Line numbers restart at 1 per section. Messages can have lines directly under heading, before or without sections.
5. ALL content lines, code blocks, tables, multiline constructs get line numbers. Blocks (code, tables) get one number; content starts on next line at zero indentation. Numbering inside code blocks = content, not coordinates.

## 1.3 Structure
1. **Summaries.** Summaries = content, not meta-commentary. Single line. Heading names topic; summary states conclusion, result, or key fact. No heading/summary redundancy.
2. **Message summary (mandatory).** Line after heading. For small messages, this IS the content.
3. **Section summary (recommended).** First line summarizes. Reader of only summaries gets essential structure.
4. **Scaling.** Depth grows with complexity. Minimum: heading + summary. Lines follow heading directly when sections unneeded. Sections when content has distinct aspects. Single-point sections need only summary.
5. **Blank-line chunking.** Blank lines within section separate logical groups. Line numbers continue across them.
6. **Tables.** Compact comparison, status, config data. Prefer when items share dimensions AND cells short values. Default to lines; reach for tables when items x properties pattern emerges. Include row number column; column numbers not required.

## 1.4 References
1. Reference earlier content by coordinate; do not restate. ALL references include concise parenthetical gloss slug.
2. **Point.** `@message.section.line` -> single line.
3. **Range.** Append `-line` for span within same section: `@3.2.1-4`.
4. **Cell.** `@3.2.5[2,3]` (cell), `@3.2.5[2]` (row), `@3.2.5[,3]` (column). `#` column + header row = structural, not indexed.
5. **Supersede.** Heading includes `[supersedes @message.section]` when replacing prior content.

## 1.5 Expand
1. User requests expansion of coordinate -> next message provides more depth. Counter increments normally; heading references expanded coordinate with gloss. Content follows ALL rules with more depth. Expansions can themselves be expanded.

## 1.6 Notation
1.
| Syntax | Meaning |
|--------|---------|
| `->` `<-` `<->` | flow; derivation; bidirectional |
| `=>` `<=>` | implies; iff |
| `(...)` | metadata, context |
| `AND` `OR` | conjunction; disjunction |
| `ALL` `ANY` `NONE` | universal; existential; empty |
| `IF`/`THEN`/`ELSE` | branching; 1-2 paths |
| `MATCH`/`CASE` | branching; 3+ paths |
| `IN` `NOT-IN` | membership |
| `MUST` `SHOULD` `MAY` | obligation (required > recommended > optional) |
| `=` `!=` `~=` | equals; not equal; approximate |
| `>` `<` `>=` `<=` | comparison |
| `+` | add, include, concatenate |
| `-` | remove, exclude, without |
| `~` | approximate |
| `△` | diff, change |
| `?` | optionality |
| `/` | paired values; ratio; related alternatives |
| `x...` | repeatable element |
| `{a, b, c}` | unordered set |
| `[a, b, c]` | ordered sequence |
| `a \| b \| c` | alternatives |
| `#scope` | domain scope |
| `[confirmed]` `[likely]` `[plausible]` `[speculative]` | confidence (verified > strong > reasonable > low) |
| `[important]` `[note]` `[question]` `[action]` `...` | immediate user attention |
| `[open]` `[wip]` `[done]` `[blocked]` | pending; in progress; resolved; blocked |

## 1.7 Examples
1.
```
# 1 Gateway Health Check
v1 gateway p99 latency 340ms; exceeds 200ms budget since 04-12

# 2 Latency Investigation
connection pool exhaustion under peak load; pool leak suspected
1. pool sized for 100 conns; peak demand ~250
2. failed conns not released; leak compounds over hours
3. circuit breaker absent; downstream timeout 30s (too high)

# 3 Migration Plan
v1 -> v2; 3 breaking changes, 2 deprecations; target 2026-05-01

## 3.1 Breaking Changes
auth + rate-limiting + payload format; ALL clients require update
1. + OAuth2 PKCE flow replaces API key auth [confirmed]
2. - X-Api-Key header no longer accepted
3. ~ rate limit 100/min -> 500/min per client; burst 50 -> 200
4. ~ request body max 1MB -> 5MB

5. [important] ALL v1 tokens invalidated at cutover
6. [done] client SDK updated to PKCE flow
7. [wip] #security token rotation policy; target 24h expiry [likely]

## 3.2 Client Tiers
3 tiers; staggered deadlines
1.
| # | tier     | deadline   | contact   | status |
|---|----------|------------|-----------|--------|
| 1 | internal | 2026-04-20 | #platform | [done] |
| 2 | partner  | 2026-04-25 | #partner  | [wip]  |
| 3 | public   | 2026-05-01 | docs      | [open] |

2. [action] partner notification scheduled 2026-04-18

## 3.3 Rollout
phased canary rollout
1. IF canary error rate < 0.1% THEN promote to 10%
   ELSE rollback -> investigate -> retry
2. #perf p99 latency budget < 200ms; current ~180ms [confirmed]

# 4 Pool Fix Deployed [supersedes @2]
pool leak patched; p99 340ms -> 145ms
1. cause: connection leak per @2.2 (unreleased-conns)
2. fix: added finally block; + circuit breaker (5 failures -> open; 30s recovery)
3. ~ pool size 100 -> 300
4. rollback via @3.3 (canary-rollout) if regression [likely]
```

2.
```
user: expand @3.3
```

3.
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
