---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D1, D6)
    summary: The default run selection (no slugs, the launchers' mode) lists the projects root and ranks runs by transcript mtime, and no Decision says how it works once the projects root, -projects and the Claude layout leave tokencount; the locator as decided carries no recency.
    fix: Add a Decision (D1 or a new D10) for the default selection. State the candidate set (each slug with an archive tag in finished, the prime left out, located through every engine's locator at <hub>/<slug>), state the recency key and its source (for example, the locator returns each session's last-modified time beside its forks, so ranking reads no transcript), and state that a candidate with no located session is skipped, not the D7 no-sessions error, so a slug whose transcripts the provider has expired does not fail the whole default report. Add an Acceptance criterion that, over a fake read set, the default picks the -last most recently changed full loom runs and skips a finished slug with no sessions. Reword the no-full-loom-runs error so it names the engines asked, not the projects directory.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D9, D2, Acceptance criteria)
    summary: D9 says the repeated-lines figure counts what it counts today, but the Claude reader scenario skips a fork's replayed parent message, which tokencount today counts as a cross-transcript repeat through its seen set; the figure drops by one per fork unless the reader counts the skip.
    fix: In D2, define the session's collapsed repeat count as the streamed lines it collapsed plus the replayed parent messages it skipped in a fork, and add to the Claude reader acceptance scenario that the skipped replay counts in that repeat count. Leave D9's wording as is once D2 says this.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7)
    summary: D7 lets a session that vanishes between locate and read count as absent with no report trace, an unbounded silent undercount the Goal's own rationale ranks below a refusal, and its examples are false premises; a /clear starts a new transcript and keeps the old one, and pair teardown never touches the Claude projects root.
    fix: In D7, drop the /clear and teardown examples and name the real case (the provider expiring old transcripts while the count runs). State the bound, either by failing the count like an unreadable session, or by counting vanished sessions per run and naming the count in the run's section beside the cutoff's left-out count. Update the matching Acceptance criterion (a session vanished after locating) to assert that trace.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 bounds the offline set by exposing no Engine, but each capability value would hold the concrete engine, which a type assertion turns back into an Engine that can start, send or stop; the stated bound is not one the type gives.
    fix: In D6, state the bound the design relies on. Either the set holds small read-only adapters around each engine whose dynamic type is not the engine, so no assertion recovers an Engine, or the set is documented as read-only by contract and tokencount type-asserts nothing on its values.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (Goal, Scope Out)
    summary: The report's -no-cutoff header line says every transcript in each slug's project directory counts, a Claude layout term, and Out keeps report output unchanged bar the D7 line, so either the Goal's no-provider-specifics rule or Out's unchanged-output rule gives.
    fix: In Scope Out, name the -no-cutoff header line as a second permitted wording change (to every located session of each slug counts), or state that its wording stays and is not a provider specific.
---

### [BLOCKING] Default run selection has no design once the projects root leaves tokencount

**Location:** `_lyx/discussion/decision-record.md` D1, D6; `tools/tokencount/count.go` `recentRuns`; `tools/tokencount/main.go` `run`.

**Issue:** With no slugs, the mode both launchers (`tokencount.sh`, `tokencount.cmd`) use, `run` calls `recentRuns(*projects, *hub, ...)`.
`recentRuns` finds its candidates by listing every directory under the Claude projects root whose name starts with the hub's encoded path, and ranks them by the newest `ModTime` of their `*.jsonl` files.
D1's locator takes one worktree path and returns sessions with their forks, and D2's reader yields entries.
Neither gives a way to enumerate a hub's worktrees or a per-session modification time.
Scope deletes the projects resolution and Out says the report measures what it does today, but no Decision says what replaces candidate discovery or the recency key.
Ranking by the last entry's timestamp would read every finished slug's transcripts in full.
A different key could also select different runs from today's.
The record also leaves the D7 interaction undecided.
Candidates drawn from archive tags would include slugs whose transcripts the provider has expired, and D7's no-sessions error would then fail the whole default report.
Today those slugs are never candidates.
The plan writer gets no decision on the tool's default path.

### [MEDIUM] D9's unchanged repeated-lines figure conflicts with the fork replay skip

**Location:** `_lyx/discussion/decision-record.md` D9, D2, the Claude reader acceptance scenario; `tools/tokencount/count.go` `countFile`; `internal/shuttleengine/claudeengine/usage.go` `transcriptTokens`.

**Issue:** Today `countFile` counts the parent session before its sub-agents.
A fork's replayed parent message hits `seen` and increments `run.Duplicates`, which feeds the "N repeated transcript lines skipped" figure in the "All runs" total line.
The acceptance criteria require the Claude reader to skip that replayed message in the fork, as `SessionUsage` does.
D2 defines the reader's repeat count as collapsed repeat records only.
So under the record as written, the figure loses one per fork, which contradicts D9's "counts what it counts today" and the fixture criterion that reproduces today's figure.

### [MEDIUM] D7's vanished-session rule is unbounded and rests on false examples

**Location:** `_lyx/discussion/decision-record.md` D7.

**Issue:** The rationale says a silent undercount is worse than a refusal.
Yet a session that disappears between locate and read counts as absent, with nothing in the report showing it.
Nothing bounds how much this can drop.
The named causes do not delete transcripts.
`claudeengine/usage.go` states that a `/clear` starts a new session ID and transcript file, and the old file stays.
No code under `internal/` removes anything under `~/.claude/projects`, so pair teardown does not either.
The real cause, Claude Code's own expiry of old transcripts, is not named.
The rubric's attack-surface rule asks each relaxed guard to state what it lets through and what bounds it, and this one states neither.

### [LOW] D6's read-only bound does not hold by type

**Location:** `_lyx/discussion/decision-record.md` D6; `internal/shuttleengines/shuttleengines.go` `SessionNamers`.

**Issue:** D6 builds engines with a zero config as `SessionNamers` does and hands out their read capabilities.
If those capability values are the `*claudeengine.Claude` itself, as `SessionNamers` hands out its namers, then `value.(shuttleengine.Engine)` recovers a full engine.
"It exposes no `Engine`, so nothing reached through it can start, open, send to or stop a run" is then a convention, not a guarantee.
The Decision should state which bound it relies on.

### [NIT] The -no-cutoff header line keeps a Claude layout term

**Location:** `_lyx/discussion/decision-record.md` Goal, Scope Out; `tools/tokencount/main.go` `cutoffNote`.

**Issue:** The header line printed under `-no-cutoff` reads "every transcript in each slug's project directory counts".
"Project directory" is Claude's layout word, so the Goal would have the wording change, and Out forbids any report change but D7's line.
The record should say which of the two rules gives way.

## Rejected

- D1/D2 cross-engine union and dedupe: dedupe keyed by engine name and message key cannot merge two engines' messages, and each engine locates under its own root, so a session is not located twice. No finding.
- D3/D4 pricing: the zero-usage rule, unknown-model unpriced, and an engine without pricing leaving nonzero usage unpriced agree with `cost.go`'s `messageCost` and with the acceptance scenarios. No path prices nonzero usage as free. No finding.
- D2 field list against the acceptance criteria: turn flags, launch-line prompt path (rendered by `claudeengine/command.go`, so the premise holds), tool calls with path or command, tool results with their call, user texts, name and repeat count cover `endsTurn`, `startsTurn`, `readMerriamStart`, `recordBase`, `ForkTally.observe` and `CountOrch`'s prompt read. No finding beyond F2.
- Sidechain handling differs between `transcriptTokens` (parent skips sidechain lines) and `countFile` (counts all), but no current main transcript under the projects root carries a sidechain line, so it is not raised.
