---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md (the verify section)
    summary: The verify section runs the tmux tier only for reedengine and reedcli, but cards 10, 12, 13 and 14 target loomcli and card 8 targets webstercli, and both packages hold tmux-tagged tests the section never runs
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-reed-unless-name-dormant.md
    summary: Card 3 changes the skip rule to match dormant strands but leaves reedengine/doc.go saying a dead or hidden match does not count, and docs/overview.md saying the skip is for a live orch strand
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-verify-wait-note.md
    summary: Cards 10 and 12 widen lyx loom status waiting beyond parent review, but contracts/specs/loom-status-spec.md (The waiting key) and docs/overview.md still describe it as a parent-review note only, and no card edits either
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-verify-timeout-findings.md
    summary: Card 9 changes webster's gate (a timed-out run is not rerun, two new report fields) and landing's Stuck reason, but edits neither websterengine/doc.go nor landingshed/doc.go, which D8 requires for each touched package and whose verify-gate text says the failure is rerun once
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-reed-resume-done-when.md
    summary: Card 5 pins only the planResumeLaunches table, so Resume's removal of a dropped strand from state, the persisted result, ResumeResult.Dropped and the new dropped envelope key are reached by no test
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/05-reed-resume-done-when.md
    summary: Card 5 adds a condition D5 does not license, that a strand some other strand names as Parent is relaunched instead of dropped, and a test row for it
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/09-verify-timeout-findings.md
    summary: Card 9 justifies the new untagged loomshed test file by saying the package's only test file is integration-tagged, but loomshed has many untagged test files and only the verify gate's own test is tagged
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/12-loom-status-shuttle-wait.md
    summary: Card 12 plans a row for no marker and no review reading as empty, which card 10 already pins in the same reviewstatus_test.go (PATTERN-test-economy)
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/plan/16-batten-respawn-scoped.md
    summary: Card 16's running-and-unconfirmed arm adopts only a live driver strand, but its DriverStrand seam has a separate retiring value and the Intent never says whether a retiring driver strand is adopted or re-spawned over, so the new re-spawn edge has an unstated bound for that case
---

### [MEDIUM] Verify section skips the tmux tier of loomcli and webstercli

**Location:** `_lyx/plan/00-overview.md`, `## verify:`

**Issue:** The section runs `go test -tags tmux` only over `./internal/reedengine/...` and `./internal/reedcli/...`.
Cards 10, 12, 13 and 14 target `internal/loomcli`, and card 8 targets `internal/webstercli`.
Both packages hold `tmux`-tagged test files that `-tags integration` does not compile: ten in `internal/loomcli` (`smoke_driverstrand_test.go`, `integration_driverbootstrap_test.go` and others, which exercise the parked-driver branch card 14 refactors) and `internal/webstercli/cli_tmux_test.go`.
The rubric flags a targeted package whose tests the section leaves unrun, and the `tmux` tag is token-free, so running it is allowed.

**Fix:** Extend the `tmux` line to `./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./internal/webstercli/...`.

### [MEDIUM] Card 3 leaves the dormant-match rule documented the old way

**Location:** `_lyx/plan/03-reed-unless-name-dormant.md`

**Issue:** `internal/reedengine/doc.go` says an add can name a strand "whose live presence skips it (AddStrandUnless)" and that "a dead or hidden match does not count", which is exactly what D4 reverses.
`docs/overview.md` says the add row's `--unless-name orch` keeps a folder-open "on a prime whose orch strand is live" from stacking claude.
D8 requires each touched package's `doc.go` to record its new behaviour, and CLAUDE.md requires docs in the same commit.
The card edits only the flag help, the verb's Long text and the vscode comment.

**Fix:** Add `internal/reedengine/doc.go` and `docs/overview.md` to the card's `**Edit:**` list and say in the Intent what each now states: any strand of that name in this worktree's state, live, dormant or hidden, skips the add.

### [MEDIUM] Cards 10 and 12 leave the documented `waiting` contract stale

**Location:** `_lyx/plan/10-verify-wait-note.md`

**Issue:** `contracts/specs/loom-status-spec.md`, section "The `waiting` key", says the key appears only while a parent-review request is open, is prefixed `parent review: `, and is absent in every other case.
`docs/overview.md` likewise says `status` carries a waiting-on-reviewer note.
D1 makes the field also report a running verify and shuttle's waits, so both statements become false.
That spec is one of the kept contract docs, and neither card, nor any other, edits it.

**Fix:** Add `contracts/specs/loom-status-spec.md` and `docs/overview.md` to card 10's `**Edit:**` list, with the Intent stating the new field: the form `<producer>: <what> running <elapsed> (<detail>)`, the three sources in their read order, and that the key is still absent when none applies.
Card 12's Intent then needs only a sentence that the shuttle kind is the second source.

### [MEDIUM] Card 9 does not update the webster and landing docs it changes

**Location:** `_lyx/plan/09-verify-timeout-findings.md`

**Issue:** `internal/websterengine/doc.go`, section "The verify gate", says a failure is "parsed from the log and rerun once"; card 9 makes a timed-out run skip the rerun and adds `VerifyGateReport.TimedOut` and `LogTail`.
`internal/landingshed/doc.go` describes the merge-in verify without the new timeout Stuck reason.
D8 requires each touched package's `doc.go` to record its new behaviour.

**Fix:** Add `internal/websterengine/doc.go` and `internal/landingshed/doc.go` to the card's `**Edit:**` list and state in the Intent that each records the timeout outcome: webster's gate fails at once with no rerun, landing's verify halts Stuck with the log path.

### [MEDIUM] Card 5's Resume glue is untested

**Location:** `_lyx/plan/05-reed-resume-done-when.md`

**Issue:** The card's tests are rows in the `planResumeLaunches` table, a private planner.
The new observable behaviour sits in `Resume` and `reed resume`: the dropped strand is removed from state and the state persisted, `ResumeResult.Dropped` reports the count, an Info line is logged, and the envelope gains `dropped`.
No test the card names reaches any of it, and the acceptance criterion is stated for `reed resume`, not for the planner.

**Fix:** Add one case to an existing tmux-tier test that already drives `Resume` or `reed resume` (`internal/reedengine/watchdog_integration_test.go` calls `e.Resume()`, and `internal/reedcli/cli_integration_test.go` drives the CLI): a strand with all done-when paths present is gone from state after resume and `dropped` is 1, and a strand with a partly missing list is relaunched.
List the chosen test file in the card's targets.
The tmux tier is already in the plan's `## verify:`.

### [LOW] Card 5 adds an unlicensed Parent condition

**Location:** `_lyx/plan/05-reed-resume-done-when.md`

**Issue:** D5 drops a non-live strand whose done-when list is non-empty and whose every path exists.
The card adds "and no other strand in state names it as `Parent`" to the drop rule, plus a test row, and restates it in the Bound.
The decision record does not license that condition and its acceptance criterion drops every such strand.
The hidden-strand skip is existing behaviour and stays.

**Fix:** Remove the Parent condition from the drop rule, the Bound and the test rows, so the card implements D5 as written.

### [LOW] Card 9 states a false premise for its new test file

**Location:** `_lyx/plan/09-verify-timeout-findings.md`

**Issue:** The card says the new untagged `verifygate_test.go` is needed "since the package's only test file is integration-tagged".
`internal/loomshed` has many untagged test files; only the verify gate's own test, `verifygate_integration_test.go`, is tagged.
The real reason is that `NewVerifyGate` passes the production `verifytree.Timeout`, so no test through it can reach a timeout.

**Fix:** Reword to say the verify gate's only test is integration-tagged and its closure passes the production timeout, so the renderer is pinned directly.

### [LOW] Card 12 repeats card 10's empty-result row

**Location:** `_lyx/plan/12-loom-status-shuttle-wait.md`

**Issue:** Card 10 already plans a row in `reviewstatus_test.go` for the empty result when no marker and no review exist.
Card 12 plans "no marker with no review is empty" in the same file: the same behaviour through the same hook, which `PATTERN-test-economy` says extends the existing test and does not repeat.
The other three rows (shuttle marker reported, verify marker ahead of it, dead shuttle marker falls through) are new behaviour.

**Fix:** Drop the empty-result case from card 12's test list.

### [LOW] Card 16 leaves the retiring-driver case of the re-spawn arm unstated

**Location:** `_lyx/plan/16-batten-respawn-scoped.md`

**Issue:** The `DriverStrand` seam reports none, live, retiring or dead, but the running-and-unconfirmed arm says only that a live driver strand or a held run lock means a driver is there.
D6 calls a retiring strand "live to reed" and `lyx loom start` replaces one, while D7 counts it as not live.
The Intent does not say whether batten adopts a running child with a retiring driver or re-spawns over it, so the one new routing edge in this card has no stated bound for that case.

**Fix:** State in the Intent that a retiring driver strand counts as live for this arm, as D6 calls it live to reed, so the child is adopted and never re-spawned over a strand someone already asked to remove, and add the retiring case as a row in the existing `innerrun_test.go` list.
