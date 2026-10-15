---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, Scope (Out, Changing what the report measures) and D7
    summary: Out counts two wording changes, but D7 makes a third by renaming the no-sessions error from the projects directory to the engines asked
    fix: In Scope's Out bullet on changing what the report measures, replace "bar two wording changes" with "bar three wording changes", and add a sub-line after the D10 no-runs line reading "A named run's no-sessions error names the engines asked in place of the project directory (D7)."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md, D2 (repeat count sentence) and D9
    summary: D2 counts the replayed parent messages a fork skips, while today's figure counts each transcript line of them, so D9's same-value claim fails for a replay spanning several lines
    fix: In D2, replace "plus, in a fork, the replayed parent messages it skipped" with "plus, in a fork, every line of a replayed parent message it skipped", and in Acceptance criteria's Claude reader scenarios replace "counts in the fork's repeat count" with "counts each of its lines in the fork's repeat count".
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md, D7 (vanished-session sentence)
    summary: D7 says no session is ever dropped from a count that succeeds, yet the same sentence has the rerun succeed without the expired session
    fix: In D7, replace "so no session is ever dropped from a count that succeeds" with "so no located session is ever left out of a count that succeeds".
---

### [LOW] Out undercounts the wording changes

**Location:** `_lyx/discussion/decision-record.md`, Scope's Out bullet "Changing what the report measures", and D7.

**Issue:** Out says the report stays as it is "bar two wording changes" and lists the `-no-cutoff` header and help text and the default selection's no-runs error (D10).
D7 makes a further visible change: a named run with no session fails "with the existing 'no sessions' error, naming the engines asked", where today `CountRun` names the project directory (`tools/tokencount/count.go`, `no sessions for %s under %s`).
Out lists the D10 error rename, which is the same kind of change, so the omission of the D7 one leaves the plan writer with a count that disagrees with the Decisions.

### [LOW] Fork replay counted per message where today counts per line

**Location:** `_lyx/discussion/decision-record.md`, D2's repeat-count sentence, D9, and the Claude reader acceptance scenario on fork replays.

**Issue:** Today a fork's replayed parent message reaches the figure line by line: `lastLinePerMessage` adds each streamed line but the last to `Duplicates`, and the surviving line adds one more as a `seen` repeat, so a replay on k lines contributes k.
D2 has the reader skip the replay and add "the replayed parent messages it skipped" to its repeat count, and D5 shares the parse with `transcriptTokens`, which drops a skipped id on every line before any collapse.
Read literally, a replay on k lines then contributes 1, and D9's "counts what it counts today" fails.
In fork transcripts under `~/.claude/projects` the replay sits on one line in every sample I checked, so the figure holds today, but the Decision and its acceptance scenario should pin the per-line count rather than leave it to how the shared parse happens to order skip and collapse.
For a replay on one line, the figure keeps today's value: the reader's skip moves the one count from tokencount's cross-transcript repeats into the session's repeat count, and D9 sums both.

### [NIT] D7's no-drop claim contradicts its own rerun

**Location:** `_lyx/discussion/decision-record.md`, D7, the sentence on a vanished session's cause.

**Issue:** The sentence says a rerun "counts without it, so no session is ever dropped from a count that succeeds"; the rerun is a successful count without that session.
The intended claim is that no located session is left out of a successful count, and the wording should say that.
The stated cause holds: `/clear` starts a new transcript and keeps the old one (`claudeengine/usage.go`'s package comment), and no production code under `internal/` removes anything under the Claude projects root.
Since a vanished session fails the count, no session is ever silently dropped, so the report needs no drop count.

## Focus settlements

- Default mode: D10 is the Decision.
  It draws candidates from archive-tagged slugs, leaves out the prime, and ranks by the newest last-modified time of each slug's own session records, forks excluded per D1.
  That matches `recentRuns`, which takes the newest `ModTime` of the project directory's top-level `*.jsonl` files.
  A slug with no located session is skipped, not failed, which keeps D7's no-sessions error for named runs only.
  Not a defect.
- D7 vanished session: settled under F3.
- D2/D9 repeated-lines figure: settled under F2.
- D6 read-only bound: it follows from the types.
  Each adapter holds its engine in an unexported field and has only its capability's methods, so a type assertion to `shuttleengine.Engine` fails, and no capability method in the set (sessions, entries, a price, instruction file names) returns the engine.
  Absence stays representable because the adapters are one per capability, which D4's and D7's engine-without-the-capability rules need.
  Not a defect.
