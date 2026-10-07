---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/completionsignal_enforcement_test.go:112
    summary: new comment lines are hard-wrapped mid-phrase, not broken at a sentence or clause boundary
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/entryobservation.go:66
    summary: added comments carry two independent clauses on one line, joined by a semicolon
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/orchengine/doc.go:74
    summary: added comments carry two independent clauses on one line, joined by a comma and a conjunction
  - id: F4
    severity: BLOCKING
    class: consistency
    location: internal/orchengine/watcher_test.go:1184
    summary: two new or renamed untagged tests neither call t.Parallel nor name the process-global state that keeps them serial
  - id: F5
    severity: LOW
    class: consistency
    location: internal/orchengine/watcher_test.go:910
    summary: substantially edited existing untagged tests stay serial with no t.Parallel and no comment naming global state
---

Review range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
Build, `go vet` (untagged and with `integration`, `tmux` and `llm`) and the untagged tests of every touched package pass.
The delete group of card 14 and card 15 holds: the `assert-no-callers` scan for `decideHandover`, `noAttachFields`, `StartAttachOnly`, `actionAttachOnly`, `attachHint` and `mustAttach` finds no remaining caller; the one remaining mention, in the generated `docs/benchmarks/test-redundancy.md`, is a recorded measurement.
No code-level defect against the plan turned up in the spots read (`sendVerified` and its confirmation, the attach removal, the watcher's reload sequence).
The findings below are the two focus areas.

### [BLOCKING] New comments hard-wrapped mid-phrase

**Location:** `internal/shuttleengine/completionsignal_enforcement_test.go:112-116`
**Issue:** The added comment lines break inside a phrase: `so the same five` / `sit under that name.` and `beside a live` / `strand, not a verdict ...`.
`PATTERN-comment-line-breaks` requires one sentence per line with a further break only at an independent-clause boundary, and calls existing fixed-column wrapping legacy, not a convention to match.
The new lines were wrapped to match the legacy block around them.
A scan of every added Go comment line across the diff found no other line that ends mid-phrase without punctuation, apart from `internal/shedadapters/burler.go:505`, which ends on a dash inside a sentence and is covered by F3.
**Fix:** Rewrite the added lines at lines 112-116 as one sentence per line, breaking only at sentence ends and independent-clause boundaries.
Leave the legacy lines around them untouched.

### [BLOCKING] Semicolon joins two independent clauses on one line

**Location:** `internal/loomcli/entryobservation.go:66`
**Issue:** `PATTERN-comment-line-breaks` requires a break at an independent-clause boundary, and a semicolon with a clause of its own after it is one.
Added comment lines that keep both halves on one line:
- `internal/loomcli/entryobservation.go:66`
- `internal/loomcli/halt_test.go:39` and `:522`
- `internal/loomcli/smoke_attachprobe_test.go:349`
- `internal/loomcli/start.go:3` and `:557`
- `internal/loomengine/anomaly.go:50`
- `internal/orchcli/start.go:2`
- `internal/orchcli/start_test.go:43` and `:147`
- `internal/orchengine/doc.go:14`, `:186`, `:190` and `:232`
- `internal/orchengine/launch.go:12`
- `internal/orchengine/state.go:98`
- `internal/orchengine/watcher.go:72`
- `internal/websterengine/doc.go:313`
- `internal/websterengine/recoverbatch.go:583`
- `internal/websterengine/strand.go:114`

The list is the output of a line scan, so a fixer sweeps every added or changed comment in the diff with the same rule instead of trusting the list as complete.
**Fix:** Break each line after the semicolon, keeping the semicolon at the end of the first line.
In the struct-field lists (`start_test.go:147`, `halt_test.go:522`) a semicolon between two field descriptions may stay when what follows is a list item, not a clause with its own subject and verb.

### [BLOCKING] Comma and conjunction join two independent clauses on one line

**Location:** `internal/orchengine/doc.go:74`
**Issue:** A comma before `and`, `so` or `but`, where what follows has its own subject and verb, is an independent-clause boundary and takes a line break.
Added comment lines that keep both clauses on one line, among others:
- `internal/burlerengine/engine.go:109`, `internal/burlerengine/verdict.go:173`
- `internal/burlerengine/engine_test.go:72`, `:74`, `:489`, `:595`, `:596` and `:597`
- `internal/loomcli/entryobservation.go:21` and `:45`
- `internal/loomcli/halt.go:78` and `:79`
- `internal/loomcli/halt_test.go:497` and `:520`
- `internal/loomcli/integration_driverbootstrap_test.go:23`
- `internal/loomcli/smoke_attachprobe_test.go:320`
- `internal/loomcli/smoke_starttail_test.go:42` and `:47`
- `internal/loomcli/start.go:567`
- `internal/orchengine/doc.go:74`, `:77`, `:187`, `:223`, `:224`, `:226`, `:227`, `:228` and `:230`
- `internal/orchengine/state.go:80` and `:140`
- `internal/orchengine/watcher.go:73`, `:657-658` and `:674`
- `internal/orchengine/watchloop.go:50`, `:97` and `:145`
- `internal/shedadapters/burler.go:504-507` and `:523`
- `internal/shedadapters/burler_test.go:897` and `:901`
- `internal/shuttleengine/attach.go:52`, `internal/shuttleengine/attach_test.go:219`
- `internal/shuttleengine/claudeengine/claudeengine.go:23`
- `internal/shuttleengine/claudeengine/session.go:70`, `:170`, `:179` and `:193`
- `internal/shuttleengine/claudeengine/startup.go:344` and `:356`
- `internal/shuttleengine/claudeengine/usage.go:97` and `:98`
- `internal/shuttleengine/doc.go:91` and `:109`
- `internal/shuttleengine/engine.go:205` and `:210`
- `internal/shuttleengine/run.go:1021`
- `internal/shuttleengine/run_test.go:908` and `:937`
- `internal/webstercli/reset_test.go:96`
- `internal/websterengine/recoverbatch.go:562`
- `internal/websterengine/recoverbatch_test.go:475`

The same line-by-line scan produced these, and the same sweep instruction applies.
A comma before a conjunction that only extends a list or a compound predicate with a shared subject (for example `prints ok ..., and the parse error ...`) does not trigger a break and is not in the list.
**Fix:** Break each listed line after the comma, so the conjunction and its clause start the next line.
Sweep every other added or changed comment in the diff with the same rule.

### [BLOCKING] New untagged tests with neither t.Parallel nor a comment naming the global state

**Location:** `internal/orchengine/watcher_test.go:1184` and `internal/orchengine/watcher_compact_test.go:196`
**Issue:** `PATTERN-test-speed` requires a test to call `t.Parallel` unless it touches process-global state, named in a comment at the test or once atop its file.
`TestWatcher_AutoCompactionReloadsPluginsThenPointer` is new and `TestCompact_CycleReloadsPluginsThenPointerNamingTheNote` is the card-13 rename of an existing test; both build their own environment (`newWatchEnv`, `newCompactEnv`), assert nothing on the logger, set no environment variable and change no directory.
Their siblings in the same files, such as `TestWatcher_AutoCompactionReloadsOnlyAFreshBoundary` and the parallel subtests at `watcher_compact_test.go:340-354`, build the same environments under `t.Parallel`, so no shared state keeps these two serial.
The other new untagged tests that stay serial name their state: `TestStart_NeverAttaches` (`TMUX` set with `t.Setenv`) and `TestRun_Send_ConfirmsSubmission` (`inputSleep`).
**Fix:** Add `t.Parallel()` as the first statement of both tests.

### [LOW] Substantially edited existing untagged tests stay serial without a reason

**Location:** `internal/orchengine/watcher_test.go:910`
**Issue:** These untagged tests were edited by more than a line each in this diff and carry neither `t.Parallel` nor a comment naming global state, though a changed test falls under the same `PATTERN-test-speed` rule:
- `internal/orchengine/watcher_test.go`: `TestWatcher_ClearCycleReloadsSkillsThenPointer` (line 910), `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep` (line 1054)
- `internal/loomcli/handoffvoucher_test.go`: `TestConsumeHandoffVoucher` (line 27), `TestObserveEntry_ConsumesHandoffVoucherIntoObservation` (line 101)
- `internal/loomcli/cli_test.go`: `TestStartAliasCommand_StaysOneCommandWithSubtreeVerb` (line 50)
- `internal/shuttleengine/claudeengine/startup_test.go`: `TestComposeSend` (line 358)
- `internal/shedadapters/burler_test.go`: `TestBurlerProducer_Gate_ProbeLiveRoundPassesGateAndMapsFailedGateIdentically` (line 898)

Their one-line-touched neighbours in the same files (for example the `TestWatcher_Resume*` and `TestCompactedSince_*` tests) are left out: the rule governs what the diff changes, and a one-line edit is not rewriting the test.
**Fix:** For each test, add `t.Parallel()` (and in the table-driven ones, to each subtest that has its own fixture) after confirming it touches no process-global state; where it does, add a comment at the test naming that state.
