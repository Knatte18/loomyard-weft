---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/slackcli/toggle.go:127
    summary: The toggle's long help, the package doc and the change description claim a pre-run refusal for a directory that is no worktree is written to the trace log, but the durable sink arms only inside a lym worktree resolved from the cwd, so that refusal reaches no log.
    fix: "In the `toggle` long help (internal/slackcli/toggle.go:127-130) say that a toggle that failed, or a refusal before it started such as a directory that is no worktree, is shown on that bar first, and that a click run in a lym worktree also writes the refusal to that worktree's trace log. In internal/slackcli/doc.go:43 drop \"its pre-run's refusals included\" from the Warn claim, or narrow it the same way. In _lym/landing/summary.md, under \"Every failed toggle is logged\", add that a click whose directory is no lym worktree logs nowhere, since the trace sink has no worktree to write in. Change no code."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:50
    summary: The blast radius names only slackcli, but the ten-second hold in reedengine.ShowToggleOutcome also applies to the RC button's refusals, since internal/orchcli/remotecontrol.go:203 calls the same function.
    fix: "In _lym/landing/summary.md, set the blast radius to `slackcli, reedengine, orchcli (the RC button's rc-on refusal)`. Under \"The message stays up long enough to read\", add that the RC button's refusal is held for ten seconds too, because it shows through the same function."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedengine/slacktoggle.go:11
    summary: The toggleMessageMillis comment calls the hold a Slack toggle's message, but every non-empty message ShowToggleOutcome shows gets it, the RC button's refusal included.
    fix: "Reword the comment above toggleMessageMillis so it describes how long, in milliseconds, a message ShowToggleOutcome shows stays on the clicking client's bar, without naming the Slack toggle as its only use. Keep the second line about tmux's default display-time."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackcli/toggle_test.go:173
    summary: No test pins the Warn record that reportToggle logs for a failed click, so deleting the logger.Warn call leaves every test green.
    fix: "In TestToggle/ARefusalBeforeTheToggleStartsIsShownOnTheBar, capture the logger with internal/testkit/logcapture, as internal/websterengine/recoverbatch_test.go does, and assert one Warn record with the message `slack: a click on a Slack toggle failed`, toggle=slack and err naming `this directory is no worktree`. If logcapture redirects process-global state, move the assertion to a non-parallel subtest and name that state in its comment."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackcli/toggle_test.go:173
    summary: No test checks that productionCLI sets showToggle before wiring, so moving the assignment back into wireProduction would nil-panic on every production click whose pre-run fails, with every test green.
    fix: "Next to ARefusalBeforeTheToggleStartsIsShownOnTheBar in TestToggle, add a subtest asserting productionCLI().showToggle != nil, with a message saying a toggle whose wiring failed must still reach the bar."
---

The diff answers both halves of issue #212's expectation.
A failed click now shows on the bar: a pre-run refusal redraws the clicked bar, and the message is held for ten seconds instead of 750 ms.
A failed click is also logged at Warn.
I checked cobra v1.10.2's `execute` order: `ValidateArgs` runs before `PersistentPreRunE`, so `args[0]` is safe in `preRun`.
A pre-run abort sets the abort flag, so `runVerb` returns early and nothing is reported twice.
The `Root-Cause:` trailer names the defect the diff removes, the silent pre-run abort and the unlogged 750 ms message.
The cause of the operator's particular failed click is still unknown, and the change description says so plainly.
The entry's "or the bar or a log shows why" does not require finding it.

### [MEDIUM] The trace-log claim does not hold for a directory that is no worktree

**Location:** `internal/slackcli/toggle.go:127`

**Issue:** `logger.Warn` sends the record to the durable sink, and `resolveSinkDirLocked` (`internal/logger/sink.go:159-170`) arms that sink only when the cwd resolves to a lym worktree.
When the pre-run fails because the click's directory is no worktree, the sink therefore opens no file.
The click line also sends stderr to `/dev/null`, so the Warn record reaches no log.
The long help names exactly that case as "written to the trace log".
`doc.go:43` says "its pre-run's refusals included", and the change description says the Warn "writes a trace file in the worktree's logs".
The bar half still holds in this case, so the goal is met; only the claims are false.

### [MEDIUM] The blast radius leaves out the RC button

**Location:** `_lym/landing/summary.md:50`

**Issue:** `reedengine.ShowToggleOutcome` has a second caller, `rc-on` at `internal/orchcli/remotecontrol.go:203`, whose refusals now also stay on the bar for ten seconds.
The change description and its `Blast radius: slackcli` name only the Slack toggle.
The behaviour change is likely welcome, but the description names less than the diff changes.

### [LOW] The hold constant's comment names the Slack toggle alone

**Location:** `internal/reedengine/slacktoggle.go:11`

**Issue:** The `toggleMessageMillis` comment calls the hold a Slack toggle's message, but every non-empty message `ShowToggleOutcome` shows gets it.

### [LOW] The Warn record for a failed click has no test

**Location:** `internal/slackcli/toggle_test.go:173`

**Issue:** The bar half of the goal is pinned, but nothing captures the logger.
The trace line is read by a person, so no test should pin its wording.
A test should still pin that a failed click logs a Warn record at all, since that is half of what the issue asks for.

### [LOW] Nothing checks that showToggle is set before wiring

**Location:** `internal/slackcli/toggle_test.go:173`

**Issue:** The harness sets its own `showToggle` (`internal/slackcli/cli_test.go:327`), and the reach report shows `wireProduction` reached by no test.
The fix depends on `productionCLI` setting the seam before wiring, and no test would notice if that moved back.

## Rejected

- code-tests' LOW on the `cmd.Name() == toggleVerb` guard having no row: the guard is pinned already. `status` takes `cobra.NoArgs`, so without the guard `args[0]` would panic in `PreRunResolvesNoWorktreeForTheRelayAndBarButDoesForEveryOtherVerb`'s `status` step, and that test would fail.
- code-correctness' F2, a hand run with a required flag missing that calls `showToggle` with empty flags: `ShowToggleOutcome` only logs a failed tmux call at Debug, and the missing-flag refusal still follows. A stray tmux call on a misused internal verb does no harm worth a branch.
- code-tests' MEDIUM severity on the untested Warn line: kept as F4, lowered to LOW. The trace line is output a person reads, and the bar half of the goal is pinned.
