---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/00-overview.md:46
    summary: Cards add top-level tests to packages already at their test budget and none edits cmd/lyx/testdata/test-budget.yaml, so go test ./... fails at the first card past a budget
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/01-deny-table.md:15
    summary: Card 1 plans a new drift test that settings_test.go TestBuildDenyNotice_MatchesInstalledDenies already covers, and card 2 adds a second notice-matches-hook row set (PATTERN-test-economy)
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-state-reader.md:42
    summary: No card pins the added decision that a transcript-fallback turn end is per reader, so the acceptance row for a later reader reading it without its task has no test
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-activity-from-stamps.md:15
    summary: Cards 4 and 10 change documented behavior (LastActivity source, a third wait label) but neither updates shuttleengine/doc.go or names the stale comments in activity.go and waitmark.go
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-claude-prober.md:17
    summary: No card documents the signal parser, the process liveness lookup or the interrupt marker in claudeengine/doc.go, which the decision record lists as a doc obligation
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/12-webster-shell-wording.md:11
    summary: Card 12 rewords webster's warning, friction line and summary heading, which the decision record does not license and the plan does not record as a post-Discussion addition
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/08-state-reader.md:45
    summary: The narrowed output-files-first rule has no carrier in the plan, so the requirement that the first consumer acting on dead applies PATTERN-completion-signal first is written nowhere a future consumer would read it
---

### [BLOCKING] New test functions push packages over their test budget

**Location:** `_lyx/plan/00-overview.md:46` (`## verify:`), with the missing edit belonging to cards 2, 3, 5, 6, 7, 8, 9, 13, 14 and 17.

**Issue:**
`PATTERN-test-speed`'s "Test budget" section caps each package at the top-level `func Test…` count pinned in `cmd/lyx/testdata/test-budget.yaml`, and `cmd/lyx/testbudget_test.go` fails `go test ./...` naming a package over its row.
It says a raise is a one-line edit in the same commit as the tests that need it.
I counted the current top-level tests (excluding `TestMain`, as `isTopLevelTest` does) against the budget rows, and the packages the plan adds tests to sit exactly at their budget:

| Package | Count | Budget | Cards that add a top-level test |
|---|---|---|---|
| `internal/shuttleengine` | 143 | 143 | 6 (`sessionstate_test.go`), 8 (`sessionread_test.go`), 13 (`waitshadow_test.go`) |
| `internal/shuttleengine/claudeengine` | 91 | 91 | 2 (new `hookcmd_integration_test.go`), 3 (`signals_test.go`), 5, 7 (`prober_test.go`, two tables) |
| `internal/shuttlecli` | 19 | 19 | 9 ("one test"), 17 (the `llm` scenario) |
| `internal/orchengine` | 122 | 122 | 14 ("one table-driven test") |

`internal/battenshed` (89/89), `internal/battencli` (56/56), `internal/websterengine` (184/184), `internal/loomcli` (178/178) and `internal/webstercli` (81/81) are also at budget, so any card there that adds a function rather than a row breaks the guard too.
No card lists `cmd/lyx/testdata/test-budget.yaml` as a target, and no card says to fold its tests into existing functions as rows.
The first card that adds a function to each package above fails the `cmd/lyx` budget guard under that card's own gate and under `## verify:`'s `go test ./...`.

**Fix:**
Add `cmd/lyx/testdata/test-budget.yaml` to the `**Edit:**` list of each card that adds top-level test functions to a package at its budget, raising that package's row by the number of functions the card adds.
Where a card's planned test is really a new row of an existing table-driven test (as `PATTERN-test-economy` prefers), say so in the card and leave the budget alone.

### [BLOCKING] Card 1's drift test repeats an existing test (PATTERN-test-economy)

**Location:** `_lyx/plan/01-deny-table.md:15` and `_lyx/plan/02-python-deny.md:33`.

**Issue:**
Card 1 plans "one table-driven row set asserting that, for every input combination the existing rows cover, the notice announces exactly the rows whose hooks the document installs".
`internal/shuttleengine/claudeengine/settings_test.go:477` `TestBuildDenyNotice_MatchesInstalledDenies` already does this over every deny combination: it reads the installed `PreToolUse` matchers back out of `buildSettings` and asserts the notice carries exactly those sentences, and is empty exactly when no deny is installed.
`prepare_test.go:188` `TestPrepare_AppendSystemPromptMatchesDenyNotice` covers the launch-line half.
After the move to a table, the existing test keeps covering the behavior; only its hand-kept `wantAgent`/`wantAsk` switch needs to read the table.
Card 2 repeats the gap: its `settings_test.go` rows "showing ... the notice carrying the python sentence exactly when the hook is installed" are the same assertion with a new row.
`PATTERN-test-economy` says a new test must cover a behavior no existing test covers, otherwise it is a row in the existing test.

**Fix:**
In card 1, state that `TestBuildDenyNotice_MatchesInstalledDenies` is generalized to iterate the table, and drop the new row set.
In card 2, add the `claude_deny_python` dimension to that same test (and keep it in `TestPrepare_AppendSystemPromptMatchesDenyNotice`, as planned) instead of separate notice rows; the hook-installed-for-each-session-kind rows belong in `TestBuildSettings_DenyToggleMatrix`.

### [MEDIUM] The per-reader transcript-fallback turn end has no test

**Location:** `_lyx/plan/08-state-reader.md:42`, `_lyx/plan/03-session-signals.md:38`, `_lyx/plan/06-state-reducer.md:36`.

**Issue:**
The post-Discussion decision "Transcript-fallback turn ends are per reader" and its acceptance row say a transcript-fallback turn end whose task's completion notification has since reached the transcript reads, in a later files-only reading, without that task (`idle-done`, `no-output` or interactive `asking` where an earlier reader recorded `busy` `background`).
Card 3 reads the outstanding tasks through `outstandingBackgroundTasks` at parse time, so the behavior follows from the design, but no test row pins it.
Card 3's table has no transcript-fallback row, card 6's reducer is pure and cannot show it, and card 8's `sessionread_test.go` list (idle-done with outputs, dead waiting pid, skipped records, prompt offset, session id, missing parser, absent root) has no row for it.
Card 16's corpus states do not reach it either, since each case replays one parse.

**Fix:**
Add a row to card 8's `sessionread_test.go` table (or card 3's `signals_test.go` table): a Stop line with no `background_tasks`, a transcript that lists a running shell, read once as `busy` `background` with `SignalTranscript`, then the same events file read again after the transcript carries the completion notification, reading `idle-done` (outputs present) with no task.

### [MEDIUM] Cards 4 and 10 leave documented behavior stale

**Location:** `_lyx/plan/04-activity-from-stamps.md:15`, `_lyx/plan/10-held-wait-mark.md:15`.

**Issue:**
`internal/shuttleengine/doc.go` states "The last activity is the newer of the transcript's last write and the events file's last write", and `AgentActivity.LastActivity` and `activity.go`'s header say the same.
Card 4 makes the events-file part the stamp of the newest Stop or ask line (with an mtime fallback), but neither `doc.go` nor those comments are in its targets or its Intent.
Likewise `doc.go` says "Wait shows its two Go-side waits ... `gate <entry name>` ... `background shells`", and `waitmark.go`'s header and `WaitMarker` comment name the same two labels; card 10 adds a third label `held`, `ReadWaitMarkers` and `Held`, and updates none of these texts.
`CLAUDE.md` requires a change to observable behavior to update the module doc in the same commit.

**Fix:**
Add `internal/shuttleengine/doc.go` to cards 4 and 10, and state in each Intent that the comments in `activity.go` (card 4) and `waitmark.go` (card 10) are reworded to the new behavior.

### [MEDIUM] claudeengine/doc.go never learns the signal parser, liveness or interrupt marker

**Location:** `_lyx/plan/07-claude-prober.md:17`, `_lyx/plan/03-session-signals.md:17`.

**Issue:**
The decision record's Scope lists `internal/shuttleengine/claudeengine/doc.go` among the same-commit docs, and that file already describes the activity reading and the registry resume check.
Cards 2 and 5 edit it (deny table; event-file line format, stamp pairing, append races), but card 3 (`ParseSessionSignals`, the signal kinds, which `SessionEnd` reasons end a process) and card 7 (`ProcessLiveness`, `TurnStartInterrupt`, the shared registry reader) add the main new `claudeengine` capabilities and neither edits `doc.go` or states a doc.
A reader of the package doc finds the stamp lines but not the parser that reads them.

**Fix:**
Add `internal/shuttleengine/claudeengine/doc.go` to card 3 (signals, session-end reasons) and card 7 (liveness from the registry, `unproven` when no entry, the interrupt marker), with the doc text named in each Intent.

### [LOW] Card 12 is scope the record does not license

**Location:** `_lyx/plan/12-webster-shell-wording.md:11`.

**Issue:**
D8 says leftover shells are "logged and returned as today's `Result.ExpiredShells`, so webster's friction note still records them", and the Notes name `runlevel.go` only as where that note lives.
Nothing in the record licenses rewording `expiredShellWarning`, renaming the `summary.md` section heading from `Background shells waited out` to `Background shells left running`, or editing `websterengine/doc.go`.
The rewording is a reasonable consequence of D8 (the old text "ran past `background_shell_wait_min`" is false for a done-at-once shell), but the plan stencil's Step 1 says a scope addition after Discussion is recorded with `lyx loom decision add --by parent` before planning on it, and the overview does not show that.
The heading rename is also an observable output change beyond what the falsehood requires.

**Fix:**
Either record the wording change as a decision addition (and keep the card), or narrow the card to the sentences that become false and leave the section heading as it is.

### [LOW] The narrowed output-files-first rule has no carrier

**Location:** `_lyx/plan/08-state-reader.md:45`, `_lyx/plan/06-state-reducer.md:31`.

**Issue:**
The added decision "Output files first covers output-derived states only" and the `PATTERN-completion-signal` constraint require that the first consumer to act on `dead` applies the output check before acting.
Card 6 states the precedence rules and card 8's `doc.go` section says "no consumer acts on it", but no card writes the handover rule down; `PATTERN-completion-signal` and "first consumer" appear in no card.
The reducer ranks `dead` and `unknown` above `idle-done` on purpose, so the next task's author has nothing in the code or docs that warns a finished run can read `dead`.

**Fix:**
Add one sentence to card 8's `shuttleengine/doc.go` section (and to the `SessionFold` doc comment in card 6): `dead` and `unknown` outrank `idle-done`, so the first consumer that acts on `dead` applies `PATTERN-completion-signal`'s output check first.
