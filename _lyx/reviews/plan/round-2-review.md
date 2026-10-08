---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/13-wait-shadow-log.md:19
    summary: "Card 13 adds the wait loop's shadow logging (state changes, disagreements, the separate signal cursor) but names no doc.go target, so internal/shuttleengine/doc.go never describes it"
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-webster-shell-wording.md:19
    summary: "Card 12 rewords expiredShellWarning but leaves out internal/websterengine/triage_test.go, whose required substrings assert the clause being reworded"
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-payload-shell-wait.md:30
    summary: "Cards 10 and 11 plan new top-level Wait-level functions for behavior that TestRun_Wait_HeldTurnEnd and TestWait_HeldTurnEndNotice already drive, and card 11 justifies its function with a claim that is false"
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/11-payload-shell-wait.md:4
    summary: "Card 11 changes what Result.ExpiredShells holds but the field comment in run.go, which says each shell outlasted background_shell_wait_min, is not among its targets"
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/02-python-deny.md:4
    summary: "Card 2 adds a third deny key but leaves the ConfigTemplate comment in template.go and the Spec.Interactive comment in spec.go, which enumerate the deny keys, out of its targets"
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/03-session-signals.md:29
    summary: "Card 3 holds consumed back only at one trailing stamp line, narrower than the added decision, which holds it at every stamp with no payload line after it"
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/plan/13-wait-shadow-log.md:22
    summary: "Card 13 gives the wait loop a signal cursor separate from run.offset but never says where it starts, and run.offset is the wrong start for an attached run that resumes at an asking offset"
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/05-hook-stamps.md:29
    summary: "Card 5 asserts exit status 1 on a failed append for Stop only, while the acceptance row names the ask record too"
---

### [MEDIUM] Card 13 changes observable wait-loop behavior with no doc.go target

**Location:** `_lyx/plan/13-wait-shadow-log.md` (Edit list and Intent); `internal/shuttleengine/doc.go`
**Issue:** Card 13 makes `Wait` log `shuttle: session state` on every state change and `shuttle: session state disagrees` on a mismatch under the D5 mapping, with three expected pairings that log nothing, on a signal cursor that is separate from `run.offset`.
That is new observable behavior of `internal/shuttleengine`.
The card's targets are `waitshadow.go`, `wait.go`, `run.go`, `fakes_test.go` and the budget row, with no `doc.go`, and its Intent has no Docs paragraph, though every sibling card that changes behavior in this package has one (cards 4, 8, 10, 11).
Card 8 lands first and writes the shadow-mode section, but it can only describe what exists then: the signals, the states, the facts, and that no consumer acts on the state.
Nothing after it records the wait loop's logging, so the decision record's "Docs in the same commit: `internal/shuttleengine/doc.go`" is met for the model and missed for this site.
CLAUDE.md puts a built design in its package's `doc.go` in the commit that lands the code.
**Fix:** Add `internal/shuttleengine/doc.go` to card 13's Edit list and give the Intent a Docs paragraph: the wait loop logs each state change and each disagreement, the fixed mapping, the three expected pairings, that the loop never branches on the state, and that the cursor stops before an unpaired trailing stamp.

### [MEDIUM] Card 12 rewords a sentence a test outside its targets asserts

**Location:** `_lyx/plan/12-webster-shell-wording.md` (Edit list, Tests paragraph); `internal/websterengine/triage_test.go`
**Issue:** The card rewords `expiredShellWarning` and lists `summary_test.go` and `runlevel_test.go` as the tests that move.
`triage_test.go` also asserts the warning and the friction note's bullet, and its required substring list includes `stopped waiting on the shell at a turn end, which finished the run when Master's output files existed and otherwise held it for the parent`, `lyx did not stop the shell` and the bound `` `background_shell_wait_min` (15 minutes) ``.
The card replaces the clause that carries the first of these ("the wait stopped waiting on the shell at a turn end, which finished the run when ... held it for the parent" next to "ran past") and says only that the bound and strand-ending clauses are kept.
Whether that substring survives is left to the implementer, so the card's gate can fail on a stale assertion in a file the card does not own.
The writer's stencil rule 4 says every test asserting a reworded message joins the card's targets.
`webstercli/verbs_test.go` (`TestRunCmd_DiedMasterNotesExpiredShellOutcome`) asserts the bound and `lyx did not stop the shell` as well, and stays green only if the card keeps those two verbatim.
**Fix:** Add `internal/websterengine/triage_test.go` to the Edit list and name it in the Tests paragraph, and state in the Intent which clauses stay byte-identical (the bound with its minutes, `lyx did not stop the shell`, the strand-ending clauses) so `webstercli/verbs_test.go` stays valid.

### [MEDIUM] Cards 10 and 11 add Wait-level test functions that existing held-turn-end tests already drive

**Location:** `_lyx/plan/10-held-wait-mark.md` (Tests, `waiting_test.go` bullet); `_lyx/plan/11-payload-shell-wait.md` (Tests, the new `Wait`-level function); `internal/shuttleengine/wait_test.go:396`; `internal/shuttleengine/hold_test.go:33`
**Issue:** `PATTERN-test-economy` sends a case to a row of the existing test of that behavior unless no existing test covers it.
Card 11 says "no existing test drives a whole wait past the bound to its next turn end or deadline".
`TestWait_HeldTurnEndNotice` does drive `Wait` past the shell bound: its rows with `shellJump` run the whole loop on a jumping clock against a `waitingEngine` with outstanding shells, and assert the notices and the outcome at the deadline.
A payload-reported shell that keeps waiting past the bound with no held turn end, no notice and `OutcomeTimeout` at the deadline is a sibling row in that table, with `Signal: SignalPayload` on the task.
Card 10 justifies its function with "`TestShellWaitMark` pins the background-shell wait only".
`TestRun_Wait_HeldTurnEnd` already drives `Wait` over held turn ends with a `fakeReed` that records `WaitMarkCalls`, through a later stop that clears the hold, a dead pane and a timeout while held, and the card's three assertions (mark set at the hold, cleared at the next event, nothing left when `Wait` returns held) are assertions on those rows.
Both cards also raise the `internal/shuttleengine` row for the added functions, which folding removes.
Parts of card 11's function, the log-once across ticks and the finish at the next turn end, have no existing row and need a harness the table lacks (it picks either `shellJump` or `script`), so those may stay a function once the deadline half is folded.
**Fix:** Fold card 10's mark assertions into `TestRun_Wait_HeldTurnEnd` and drop its new function and budget raise.
For card 11, add the payload-shell deadline case as a row of `TestWait_HeldTurnEndNotice`, keep a new function only for the log-once and next-turn-end behavior, and correct the "no existing test" sentence.

### [LOW] Card 11 leaves the Result.ExpiredShells field comment describing the old behavior

**Location:** `_lyx/plan/11-payload-shell-wait.md` (Edit list); `internal/shuttleengine/run.go:213`
**Issue:** The comment reads "ExpiredShells holds the labels of the background shells this run waited out, in expiry order: each outlasted background_shell_wait_min and was let through as a turn end."
After card 11 a gated run that finishes Done at once also lists its payload-reported unawaited shells there, without any shell having outlasted the bound.
Card 12 admits `ExpiredShells` now holds two kinds of shell and rewords webster's text, but `run.go` is in no target list until card 13, which edits it for another reason.
The card's Docs paragraph names `template.yaml`, `config.go`, `wait.go` and `doc.go`, not the field comment.
**Fix:** Add `internal/shuttleengine/run.go` to card 11's Edit list and give the Docs paragraph the field comment: the labels are the shells the wait stopped waiting on, which includes payload-reported shells a gated run's turn end left behind at once.

### [LOW] Card 2 leaves two comments that enumerate the deny keys stale

**Location:** `_lyx/plan/02-python-deny.md` (Edit list); `internal/shuttleengine/template.go:17`; `internal/shuttleengine/spec.go:84`
**Issue:** `ConfigTemplate`'s doc comment lists the template's plain-literal keys as `claude_deny_agent_tool, claude_deny_ask_user_question`, and card 2 adds `claude_deny_python` as another.
The `Spec.Interactive` comment says the Agent deny is included in both modes and each deny is toggleable through `claude_deny_agent_tool` / `claude_deny_ask_user_question`, which is no longer the whole set once the python deny is installed in both modes.
Card 2 edits `config.go` and `template.yaml` but neither of these files.
**Fix:** Add `template.go` and `spec.go` to card 2's Edit list, with the two comments updated to name the python key and deny.

### [LOW] Card 3's trailing-stamp rule is narrower than the added decision

**Location:** `_lyx/plan/03-session-signals.md` (Intent, `SessionSignalParser` bullet and Tests paragraph)
**Issue:** The added decision "Incremental reader holds a trailing stamp" says an incremental reader does not advance its offset past a stamp line with no payload line after it.
Card 3 says `consumed` "stops before a trailing stamp line", singular, and its test row has one trailing stamp.
Two hooks of one session can write their stamps before either payload (a stamp of one hook and a stamp of another, then the payloads).
A poll that lands there sees two stamps with no payload after them.
Holding back only the last one advances past the first, so when its payload arrives at the next poll the stamp is already consumed, the payload reads a zero `At`, and the reader's `since` differs from a whole-file read, which breaks the acceptance row for that case.
**Fix:** Define `consumed` as stopping before the earliest stamp line of the trailing run of stamp lines that has no payload line after it, and add a row with two trailing stamps of different hooks to the existing `signals_test.go` table.

### [LOW] Card 13 leaves the signal cursor's starting offset open

**Location:** `_lyx/plan/13-wait-shadow-log.md` (Intent, first bullet of `Run` gains)
**Issue:** The card gives `Run` "a signal cursor separate from `run.offset`" and does not say where it starts.
The reducer folds only signals past the run record's `promptOffset` (D3).
`run.offset` starts at the prompt offset on a fresh run but starts at a recorded asking offset on an attach that resumes there (`attach.go`, `startOffset`), so a cursor seeded from `run.offset` would skip turn starts and turn ends the fold needs, and the wait loop's state would differ from `lyx shuttle state` for the same file.
**Fix:** State that the cursor starts at the run record's `PromptOffset` on every path, attached or not.

### [NIT] Card 5 pins the failed-append exit status for Stop only

**Location:** `_lyx/plan/05-hook-stamps.md` (Tests, `hookcmd_integration_test.go` bullet)
**Issue:** The acceptance row says Stop and the ask record exit 1 as today when the append fails.
The card asserts "each new hook exits 0 and Stop exits 1 when the events file's directory is missing" and does not name the ask record, whose command now differs from Stop's by its stamp literal.
**Fix:** Add the ask record to the exit-1 assertion, as a row of the same function.

## Focus verdicts

- Test budget: every card that adds a top-level test function raises the owning package's row in the same card, and each stated increment matches the functions the card plans (`internal/shuttleengine/claudeengine` +1 in cards 2, 3 and 5 and +2 in card 7, `internal/shuttleengine` +1 in cards 6, 10, 11 and 13 and +2 in card 8, `internal/shuttlecli` +1 in cards 9 and 17, `internal/orchengine` and `internal/battenshed` +1 in cards 14 and 15). The cards that add none (1, 4, 12, 16, 18) fold their tests into rows of named existing functions. The base rows equal the current counts, so nothing is off. F3 asks to fold two of those functions into existing ones, which would lower the `internal/shuttleengine` raise.
- Deny cards: card 1 generalizes `TestBuildDenyNotice_MatchesInstalledDenies` and card 2 adds the python switch as a dimension of it and of `TestPrepare_AppendSystemPromptMatchesDenyNotice` and `TestBuildSettings_DenyToggleMatrix`. No card plans a new notice-matches-hooks test, so there is no finding. The one new function is the real-shell round trip, which the decision record puts in the `integration` tier and no existing test covers.
- Same-commit docs: findings F1, F4 and F5 are the gaps, for cards 13, 11 and 2. Cards 3, 4, 5, 7, 8, 10 and 11 carry `doc.go` and file-comment updates in their targets or Intent, and card 1 changes no behavior.
- Decision-record coverage: every post-Discussion decision and acceptance row is carried by a card, and each card that plans tests pins it. The `idle_prompt` acceptance row that D1 still states is superseded by the added decision, and card 4 follows the later decision and says so in the Shared Decisions. The two gaps found are F6 (the trailing-stamp hold) and F8 (the ask record's exit status).
