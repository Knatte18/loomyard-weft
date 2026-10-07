---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/11-watcher-reloads-fresh-compaction-only.md
    summary: A boundary held in w.compactedAt from an earlier tick still reaches tickIdle after a later tick finds two turn ends or none, so the watcher types a reload D8 forbids
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/13-orch-reload-sequence.md
    summary: The plugins step is never carved out of tickResuming and typeReloadStep, so a restart at that step is read as a skills step and skips the reload
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-loom-start-never-attaches.md
    summary: Two tmux-tier tests in smoke_test.go assert the tmux handover output and are not in the card, so the plan's own tmux verify run goes red
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/15-orch-start-never-attaches.md
    summary: PATTERN-cli-cobra still lists lyx orch start as an interactive-handoff exception after the card removes the handoff
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-rerun-removes-superseded-strand.md
    summary: The strand teardown sits in the not-found return that probe-only callers also reach, and the card states no bound for those callers
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-send-confirms-submission.md
    summary: The acceptance criterion names a running turn as a case that sends no extra Enter, and neither the reader table nor the send table has that row
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/04-escalation-halt-skips-reflection.md
    summary: Cards 1 to 7 change observable CLI behavior but name no module doc, so shuttleengine/doc.go, websterengine/doc.go and docs/overview.md line 257 go stale
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/plan/08-step-error-keeps-handoff-voucher.md
    summary: The card's fallback fix sites and the loomPostStep comment live in arm.go, which the Edit list omits
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/16-orch-start-waits-for-stopping-watcher.md
    summary: The WaitWatcherGone unit rows for release within the bound and live past the bound duplicate the two start_test rows that cover the same behavior
  - id: F10
    severity: LOW
    class: design
    location: _lyx/plan/18-burler-validate-review-verb.md
    summary: The verb ok and error test is a subset of the parity test table, so the same behavior is pinned by two new tests
  - id: F11
    severity: NIT
    class: decision
    location: _lyx/plan/02-claude-input-box-reader.md
    summary: submitRedrawSettle is left as a few hundred milliseconds while its sibling submitSettleMS is pinned at 300
---

### [BLOCKING] A held compaction boundary survives a later tick that finds it stale

**Location:** `_lyx/plan/11-watcher-reloads-fresh-compaction-only.md`, the `Watcher.tick` rules.
**Issue:** The card holds a confirmed boundary (`TurnEndsAfter == 1` and `ReadTurnEndAfter`) in `w.compactedAt` "as today", so `tickIdle` enters the reload once the idle probe passes.
In `internal/orchengine/watcher.go`, `tickIdle` starts `startAutoReload` whenever `w.compactedAt` is non-zero, and nothing but `startAutoReload` clears it.
Suppose the idle probe fails on the tick that held the boundary, for instance while the box holds a draft, and the session ends more turns before the probe passes.
The next tick evaluates the same boundary with two turn ends after it.
The card's first rule says to move the baseline and "type nothing", and its third rule says the same for the remaining cases.
Neither rule drops the held value, so `tickIdle`, which runs right after in the same tick, still sees a non-zero `w.compactedAt` and types the reload.
That is the reload "typed after turns that ran without it" that D8 and its acceptance criteria forbid.
**Fix:** State in the card that every tick which reads a turn end re-evaluates the boundary and replaces the held value, and that rules 1 and 3 clear `w.compactedAt`.
Add a row to the watcher test: a held boundary whose idle probe fails, followed by a tick that reads a second turn end, types nothing and moves the baseline.

### [MEDIUM] The plugins step is not carved out of the resuming tick

**Location:** `_lyx/plan/13-orch-reload-sequence.md`, the Watcher section.
**Issue:** The card names `reloadStep` and `startReload`, and says the plugins step persists `ReloadTypedAt` before typing.
It does not name `tickResuming` or `typeReloadStep`.
`tickResuming` treats every step other than the pointer as a skills step: once `ReloadTypedAt` is set, a turn end read after a restart goes to `settleSkillTurn`, and a step older than the handoff timeout skips `w.skills` with the cause `timeout` and advances to the pointer.
`typeReloadStep`'s `switch` sends the pointer for any step it does not list.
A watcher restarted at the plugins step therefore skips `/reload-plugins` and the skills step, which is the restart case D10 requires to resume at the plugins step.
**Fix:** Add to the card that `tickResuming` handles the plugins step before its skills branch: no typed, timeout or settle logic, only the idle probe and a retype.
Add that `typeReloadStep` gains a plugins case calling `Session.ReloadPlugins`.
Extend the restart test so the persisted `ReloadTypedAt` is older than the handoff timeout and a turn end has been read.

### [MEDIUM] Card 14 misses two smoke tests that assert the handover

**Location:** `_lyx/plan/14-loom-start-never-attaches.md`, the Tests list and the `ImpactSummary`.
**Issue:** `TestSmokeBootstrap_MalformedStatusProceedsToHandoverAndLogsWhy` and `TestSmokeBootstrap_DiedDriverProceedsToHandoverAndLogsWhy` in `internal/loomcli/smoke_test.go` run `lyx loom start` without `--no-attach` and require `not a terminal` on stdout as evidence the tmux handover was reached.
Both fail once the handover is gone, and `go test -tags tmux ./internal/loomcli/...` in the plan's `verify:` runs them.
The comments in `bootstrap.go` (`dispositionForHandshake` and the handshake constants) and in `integration_driverbootstrap_test.go` still describe a terminal handover.
The `ImpactSummary` says only help, docs and the envelope's two keys change.
**Fix:** Add `internal/loomcli/smoke_test.go` to the Edit list with the two tests moved to assert the success envelope and the unchanged driver-log and lock assertions, and rename them to drop "Handover".
Add the stale `bootstrap.go` and `integration_driverbootstrap_test.go` comments to the card's text, and correct the `ImpactSummary`.

### [MEDIUM] PATTERN-cli-cobra still lists lyx orch start

**Location:** `_lyx/plan/15-orch-start-never-attaches.md`.
**Issue:** `pattern/PATTERN-cli-cobra.md`'s interactive-handoff exception list names `lyx orch start`.
Card 14 drops `lyx loom start` and `lyx start` from that list, and Card 15 removes the orch handoff without touching the file.
D13 requires every doc that describes the start verbs' terminal handoff to change in the same commit.
**Fix:** Add `pattern/PATTERN-cli-cobra.md` to Card 15's Edit list with the sentence dropping `lyx orch start` from the list.

### [MEDIUM] Probe-only callers of Attach inherit the teardown unbounded

**Location:** `_lyx/plan/05-rerun-removes-superseded-strand.md`, the Intent and the Bound.
**Issue:** The card puts the removal in the not-found return of `AttachGated`, and names SingleLLM, Burler's `probeLiveRound` and the Bouncer as its callers.
Other callers use the same return as a pure probe that starts no run afterwards: the Bouncer's `awaitLiveJudge` and `awaitLiveSeed` (whose doc says "a probe that never spawns anything") and the reflection probe in `internal/frictionengine/reflect.go`, whose report-exists branch archives without a spawn.
Each would now remove a live strand of a respawn-eligible run on a not-found answer.
D4's rationale places the rule in the probe's respawn branch, so the placement is licensed, but the card states no bound for these callers and no test reaches one of them.
**Fix:** State in the Intent that every caller whose probe answers not found inherits the teardown, and why it is safe there: the strand belongs to a run the probe judged not live, and the output files were never consumed by it.
Add the probe-only callers to the Bound, and add a row to the `attach_test.go` table over the plain `Attach` entry point.

### [MEDIUM] The running-turn case has no test row

**Location:** `_lyx/plan/03-send-confirms-submission.md`, the test table; `_lyx/plan/02-claude-input-box-reader.md`, the reader table.
**Issue:** The acceptance criteria say a send whose box holds "a draft of other text, including ... a collapsed paste placeholder or a running turn" sends no extra Enter.
Card 3's table lists the draft, the longer draft, the placeholder and no box, but no running turn.
Card 2's reader table omits `testdata/pane-turn-running.txt`, whose capture carries `esc to interrupt` over an empty box.
**Fix:** Add the running-turn row to both tables: the reader answers ok with empty text for that capture, and the send answers success with no extra Enter.

### [LOW] Cards 1 to 7 name no module doc

**Location:** `_lyx/plan/04-escalation-halt-skips-reflection.md`, and Cards 1, 2, 3, 5, 6 and 7.
**Issue:** The repo's CLAUDE.md requires a task that changes observable CLI behavior to update the module doc in the same commit.
Cards 11, 13, 15, 16 and 18 do.
Cards 1 to 7 do not, and three docs go stale: `internal/shuttleengine/doc.go` states what `Runner.Attach` answers and where the verified send path sits; `docs/overview.md` line 257 says loom runs the reflection "after a `blocked` or `failed` halt"; `internal/websterengine/doc.go` describes the recovery strand's lifecycle through `recover-batch`.
**Fix:** Add `internal/shuttleengine/doc.go` to Cards 3 and 5, with a paragraph on the box-confirmed send and one on the teardown in the not-found return.
Add `docs/overview.md` to Card 4 for line 257, and `internal/websterengine/doc.go` to Cards 6 and 7 for the strand removals.

### [LOW] Card 8's fallback and the loomPostStep comment are in arm.go

**Location:** `_lyx/plan/08-step-error-keeps-handoff-voucher.md`.
**Issue:** The card's fallback names "a step path that bypasses `PostStep`" and "a history entry appended after `loomPostStep` recorded the voucher" as candidate causes, and `loomPostStep` and `loomPreStep` are in `internal/loomcli/arm.go`, which the Edit list omits.
`loomPostStep`'s doc comment says the voucher is recorded after a successful `shed.Step`, and the card adds a second recording site on the error path.
**Fix:** Add `internal/loomcli/arm.go` to the Edit list and name `loomPostStep`'s comment in the doc updates.

### [LOW] Card 16 pins the wait twice

**Location:** `_lyx/plan/16-orch-start-waits-for-stopping-watcher.md`, the Tests list.
**Issue:** The `WaitWatcherGone` rows "returns not live once a lock holder releases within the bound" and "returns live past the bound" cover the same behavior as the two `start_test.go` rows, which reach `WaitWatcherGone` through the cobra verb with an injected `sleep`.
Only "probes exactly once with no stopping record" is not reached by those rows.
**Fix:** Keep the stopping-record tests and the probe-once row in `watchloop_test.go`, and cut the two duplicate rows.
Have `TestStart_LiveStrandAndWatcherDoesNothing` assert the injected `sleep` is never called, so the healthy path is pinned at the verb too.

### [LOW] Card 18 tests the verb twice

**Location:** `_lyx/plan/18-burler-validate-review-verb.md`, the Tests list.
**Issue:** The first test (exit 0 on a valid file, exit 1 with `ParseReview`'s error and the quoting hint on an invalid one) is a subset of the parity test, which runs the verb over valid, invalid YAML, malformed quoted summary, missing delimiter and missing file.
**Fix:** Fold the exit-code and quoting-hint assertions into the parity table's rows and drop the separate test.

### [NIT] Card 2 leaves one settle value open

**Location:** `_lyx/plan/02-claude-input-box-reader.md`.
**Issue:** Card 1 pins `submitSettleMS = 300`; Card 2 gives `submitRedrawSettle` only as "a few hundred milliseconds", so the implementer chooses the value.
**Fix:** Pin the value in the card.
