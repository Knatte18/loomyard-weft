---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decision 8
    summary: Status is now read for named slugs, so a named in-flight slug silently gets an older run's archive tag as its status, and Decision 8 states no bound for that edge
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md Decision 1
    summary: A local-command user line typed while idle, such as /model, does not end a turn, so the gap after it counts as working time
  - id: F3
    severity: NIT
    class: decision
    location: _lyx/discussion/decision-record.md Decision 8
    summary: The foreign -hub refusal does not say how the two directories are compared, so the same hub spelled with a trailing slash or a relative path is refused
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks
    summary: The mid-turn crash sentence appears in both Auto-mode assumptions and Open risks
---

### [MEDIUM] A named in-flight slug reads an older run's status without a bound

**Location:** `_lyx/discussion/decision-record.md` Decision 8, with the matching bullet under Open risks.

**Issue:** Decision 8 adds a routing edge.
Until now, named slugs read no archived status.
Now every named slug reads the newest `archive/<slug>/<sha>` tag.
The package doc in `tools/tokencount/main.go` says a named run may still be in flight.
If an older finished run used the same slug, the in-flight run gets that older run's tag.
Decision 3 then takes the older seed's `started_at` and the older history's last end as the span.
Decision 2 clips the role times to that span, and Decision 5 prints the older run's Producers table.
The section's header names the in-flight slug and says nothing is wrong.
Its numbers come from the older run and the in-flight transcripts that fall inside the older span.
Open risks names this case, but the attack-surface rule wants the bound in the Decision itself, and Decision 8 has none.
The round-1 fixer report says the newest-tag rule cannot bound this case.
That is true of tag choice alone, but the tool has another signal.
Its own doc defines a run as finished once its pair is torn down, and teardown removes the worktree, so a run whose worktree still exists is in flight.

**Fix:** In Decision 8, state what this edge lets through and give it a bound.
For example: when the named slug's worktree `<hub>/<slug>` still exists, the run is in flight.
Its pair is not torn down, so the tool reads no archive tag, and the section uses the transcript-window fallback with "no shed status was read".
Then add an acceptance criterion for that case and drop the Open risks bullet, or keep it only for whatever risk is left.

### [LOW] Local-command lines start working time when typed while idle

**Location:** `_lyx/discussion/decision-record.md` Decision 1.

**Issue:** Under Decision 1, every user line that is not the interrupt marker starts working time until the session's next line.
Claude Code also writes user lines for local slash commands: `<command-name>…`, `<local-command-caveat>…` and `<local-command-stdout>…`.
None of these starts a model turn.
I checked the task-worktree transcripts under `~/.claude/projects`.
In `-home-knatte-Code-loomyard-LYXHUB-gate-test-scope/aa43a0aa-….jsonl`, a `<local-command-stdout>Set model to Haiku 5.5 …` line is followed 677 s later by the operator's next prompt.
Decision 1 counts all 677 s as working time.
In the prime's transcripts, the same pattern runs up to 8427 s after a `Cancelled` stdout line.
Those transcripts are orch and fall out under Decision 4, but they show how large the gap can get.
I confirmed the interrupt-marker case the focus asks about against a real transcript.
In `-home-knatte-Code-loomyard-LYXHUB-test-suite-prune-rest/bd330a6d-….jsonl`, the `[Request interrupted by user]` text block comes right after a tool result.
The next line is 43 minutes later, and Decision 1 correctly makes that gap idle.
The `stop_sequence` stop reason also shows up, on a synthetic API-error line, and correctly ends the turn.

**Fix:** Add a rule to Decision 1: a user line whose text starts with `<local-command-stdout>` ends the turn.
`/compact` writes its command line first and its stdout line last, so the compaction between them still counts as work, and the idle gap after a `/model` stays idle.
Add an acceptance criterion: a local command typed between turns adds no working time.

### [NIT] The foreign `-hub` comparison is unspecified

**Location:** `_lyx/discussion/decision-record.md` Decision 8, "when `-hub` names a directory other than the current worktree's parent".

**Issue:** The Decision answers focus 2 in its own text.
A foreign `-hub` with an empty `-weft` is refused and the refusal names `-weft`, which bounds the wrong-hub read.
The no-slug path, which today reads the current hub's tags against a foreign hub, is covered too.
What the Decision leaves out is how the two directories are compared.
`-hub ~/Code/loomyard-LYXHUB/` (with a trailing slash), or `-hub ..` from the prime, names the current hub.
A plain string comparison against `filepath.Dir(wd)` would refuse both.
The refusal names the way out, so this is a nuisance, not a leak.

**Fix:** State that both sides are compared as absolute, cleaned paths.

### [NIT] One sentence appears twice, under Auto-mode assumptions and Open risks

**Location:** `_lyx/discussion/decision-record.md`, the first Auto-mode assumptions bullet and the second Open risks bullet.

**Issue:** "A session that crashes mid-turn … counts its gap until its next line as working" appears in both sections in nearly the same words.

**Fix:** Keep it under Auto-mode assumptions, where it is decided, and delete the Open risks copy.
