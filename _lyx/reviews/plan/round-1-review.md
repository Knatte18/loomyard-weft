---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-shedengine-relaunch.md:24
    summary: Relaunch adds shed refusals (done run, running run, missing status file) with no refusal-spec row, and TestRelaunch reaches no missing-status branch, against PATTERN-refusal-way-forward
    fix: In card 2, add `contracts/specs/refusal-spec.md` under `**Edit:**` and add two shed-section rows, `relaunch on a done run` and `relaunch on a running run`, each a correctness halt quoting the way forward Relaunch's message names; widen the trigger of the existing `status file missing` row to name `step --arbiter-attempt` beside `step` and `goto`; add a `TestRelaunch` row for an absent status file (`Admit` not called, error naming the way forward). Alternatively drop the `done` and `running` checks from Relaunch, since the caller's `Admit` stop-key check already refuses both states (an attempt's stop key is never `done` or `running`), and add only the missing-status row and test.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/14-driver-arbiter-branch.md:29
    summary: The driver's wake rule drops D6's condition of an unapplied attempt for the current stop key, and its trigger (unparked, no loop job, no arbiter job) matches every ordinary loop-job exit
    fix: In card 14, replace the wake bullet with one bound to D6's condition - re-run the arbiter command only when the driver's own last action for the current stop was running that command and its job ended without an envelope (as after a compaction or a driver revive mid-arbiter), never on the wake of the loop job's own exit; state that a refusal from that re-run falls back to the stop's existing handling under `## Stopping on a non-running state` or `## Error and interrupted stops`, never to a park quoting the refusal. Add a `wordingClaims` claim pinning the narrowed condition.
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/plan/12-relaunch-consume-attempt.md:36
    summary: Two new refusals of the consumption, a non-positive or non-numeric attempt number in loomConsumeAttempt and an unparseable verdict in consumeArbiterAttempt, are reached by no named test, against PATTERN-refusal-way-forward
    fix: In card 12, add an `unparseable verdict` row to the `arbiterconsume_test.go` table (status unchanged, no applied mark), and name a test that reaches `loomConsumeAttempt`'s bad-number refusal, either rows `0`, `-1` and `x` in that same table driving `loomConsumeAttempt` on a receiver with a temp location, or rows in card 3's waiter test through a hook that is `loomConsumeAttempt`.
  - id: F4
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-arbiter-verb.md:35
    summary: The verb's new refusal for a missing status file, an unreadable envelope or a status read error has its own refusal-spec row but no named test row, against PATTERN-refusal-way-forward
    fix: In card 10, add rows to the spawn-path test of `arbiter_test.go` for a missing status file and an unreadable envelope path, each asserting the refusal's way forward and that nothing is spawned and no attempt directory is written.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-relaunch-consume-attempt.md:23
    summary: Consumption reads the agent's verdict.yaml, so a resume the verb downgraded to hand-on for a missing precondition (no goto target at a blocked stop, no circling decision) is still consumed, and the card's Bound claim of a validated attempt is false
    fix: In card 12, make `consumeArbiterAttempt` read the attempt's `envelope.json` (the verb's validated result, card 10 step 6) and refuse unless it exists with `verdict` `resume` and an empty `cause`, taking the goto target from it; add a test row where `verdict.yaml` says `resume` but `envelope.json` reads `hand-on` with cause `precondition` (refused, no mark, status unchanged). In card 5, add the envelope type with its writer and reader (`plan:internal/loomengine#ArbiterEnvelope`, `#WriteArbiterEnvelope`, `#ReadArbiterEnvelope`) and a round-trip row in `arbiter_test.go`; card 10 writes `envelope.json` through the writer and card 12 lists the reader under `**Uses:**`.
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-arbiter-verb.md:40
    summary: The re-run path waits through AttachIfLive with Timeout the time left to the marker deadline, which is zero or negative once the deadline has passed; shuttle refuses a negative Timeout and reads zero as the config default, and the card names no branch for either, leaving a live arbiter unstopped
    fix: In card 10 step 2, add a branch before the attach - when the marker's deadline has passed, call `StopStrand` with the marker's guid and read `hand-on` with cause `timeout`, without calling `AttachIfLive` - and state that an `AttachIfLive` error reads as `hand-on` with cause `died` after the same `StopStrand`; add re-run test rows for a marker past its deadline (`StopStrand` called, no attach) and an attach error.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/14-driver-arbiter-branch.md:25
    summary: Card 14 adds driver routing edges (a re-launch over a spent repair cap, the reversal of the blocked-never-repaired rule, the arbiter run itself) and names what they skip, but its Intent states no bound
    fix: Add a `Bound:` sentence to card 14's `**Intent:**` - the arbiter run is bounded by Go's eligibility refusal and the `arbiter_max_spawns` budget (cards 9, 10), and the re-launch moves the run only through the consumption of card 12, at most once per attempt (applied mark), only backward, and never over a human's goto or pause.
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-arbiter-verb.md:52
    summary: The verb's spawn-path test asserts that each eligible stop spawns and an ineligible one spawns nothing, re-covering the per-kind table card 9 plans in arbitereligibility_test.go for the unexported helper arbiterEligibility
    fix: Per PATTERN-test-economy, test the helper through its one surface - drop `arbitereligibility_test.go` and its test-budget raise from card 9 and move its rows (each eligible and refused kind, the Publish halt with and without a target, the awaiting stop with a parent_notice but no escalation) into card 10's spawn-path table, which already asserts spawn or no spawn and the refusal's way forward. Or keep card 9's table and narrow card 10's spawn-path rows to one eligible and one ineligible stop.
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/12-relaunch-consume-attempt.md:36
    summary: The consumption test's success and Admit-refusal rows re-assert Relaunch's own goto effects (pause_before kept, status unchanged on no target or an Admit error) that card 2's TestRelaunch already pins
    fix: In card 12, have the success rows assert the applied mark and the run's current producer only, and the refusal rows the refusal message and the absent mark, leaving pause-condition preservation and the unchanged file to `TestRelaunch`.
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/plan/03-shedverbs-arbiter-attempt-flag.md:32
    summary: The new waiter test's refusal rows (nil hook, busy hook error, other hook error, each with no child) repeat the shape TestLoop_RefusalsBeforeAnyStep in internal/shedverbs/loop_test.go already covers
    fix: In card 3, add the three hook refusal rows to `TestLoop_RefusalsBeforeAnyStep` (with `internal/shedverbs#TestLoop_RefusalsBeforeAnyStep` under `**Edit:**`) and keep the new test for the paths no refusal test covers, the flag without `--until-stop` and the consumed-then-looped success row.
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/plan/02-shedengine-relaunch.md:18
    summary: Goto checks the unknown target, then the running state, then admission, so folding both target checks into one CheckGotoTarget call changes which refusal a running run with an unknown target gets, despite the card's byte-identical claim
    fix: In card 2, say where Goto calls `CheckGotoTarget`, after its running check, and state the one precedence change (a running run with an unknown target now gets the running refusal); or keep Goto's unknown-target check inline before the running check and have `CheckGotoTarget` repeat it harmlessly.
---

### [BLOCKING] Relaunch's new refusals have no refusal-spec row

**Location:** `_lyx/plan/02-shedengine-relaunch.md:24`

**Issue:** `Relaunch` refuses a missing status file, a `done` run ("relaunch never re-opens a finished run") and a `running` run with new messages, and they reach the driver through `step --arbiter-attempt` (card 3 reports them as `kind: bootstrap`, message verbatim).
Card 2 does not edit `contracts/specs/refusal-spec.md`, and card 12's three rows cover only the consumption's own refusals.
The existing `goto on a done run` and `goto on a running run` rows (refusal-spec.md:193-194) name `goto` and quote goto's way forward, not these.
`TestRelaunch` reaches `done` and `running` but not the missing file.
PATTERN-refusal-way-forward requires a row and a reaching test for each new shed refusal.

### [BLOCKING] The wake rule drops D6's condition and fires on every loop exit

**Location:** `_lyx/plan/14-driver-arbiter-branch.md:29`

**Issue:** D6 re-runs the verb only on a wake that finds "an unapplied arbiter attempt for the current stop key on disk".
Card 14's bullet keys on "unparked, with no loop job and no arbiter job running", which is also the state right after the loop job exits with its envelope, one of the wakes the stencil names (shed-template-driver.md:118).
Read literally, the driver runs the arbiter at every stop before its existing handling.
That spawns an arbiter at an error stop before or instead of the fork, the stencil-only half of D2 that card 14's own "never at an error stop whose fork named a repair verb" exists to bound.
At an ineligible stop the verb refuses, and the hand-on bullet then parks with the refusal verbatim instead of the envelope's reason.

### [BLOCKING] Consumption refusals with no reaching test

**Location:** `_lyx/plan/12-relaunch-consume-attempt.md:36`

**Issue:** `loomConsumeAttempt` refuses an attempt number that is not a positive integer, and `consumeArbiterAttempt` refuses an unparseable verdict.
Both fall under the new `step --arbiter-attempt: attempt unusable` row.
The test drives only `consumeArbiterAttempt`, and its rows are a missing verdict, a `hand-on` verdict, already applied and the admit refusals, so neither branch is reached.
PATTERN-refusal-way-forward requires a test that reaches each new refusal.

### [BLOCKING] The verb's unreadable-input refusal has no test row

**Location:** `_lyx/plan/10-arbiter-verb.md:35`

**Issue:** Step 1 refuses a missing status file, an unreadable envelope and a status read error, and card 10 adds its own `arbiter: status or envelope unreadable` row.
It does not join a grouped transient row, so the rubric's raw-I/O exemption does not apply, and a missing status file is not a raw I/O error.
None of the three test tables names a row for it.

### [MEDIUM] Consumption trusts the agent's verdict file, not the verb's validated result

**Location:** `_lyx/plan/12-relaunch-consume-attempt.md:23`

**Issue:** Card 10 step 5 reads a `resume` as `hand-on` (cause `precondition`) when a `blocked` stop's verdict names no goto target or an awaiting stop has no circling decision, and records that result in `envelope.json`.
`verdict.yaml`, written by the agent, still says `resume`.
Card 12 decides from `verdict.yaml`, and `Relaunch` checks a target only when one is named, so `step --arbiter-attempt N` on such an attempt marks it applied and steps a `blocked` run in place.
That is the reversal of the blocked-never-repaired rule without the goto target D6 makes its precondition.
The driver stencil only runs the `relaunch` line the verb emits, but the card's Bound ("a `resume` attempt the arbiter verb wrote and validated") is a Go bound and is not true as specified.

### [MEDIUM] Past-deadline re-run is unhandled

**Location:** `_lyx/plan/10-arbiter-verb.md:40`

**Issue:** A driver revived after the marker's deadline, with the arbiter strand still live (card 16 stops only dead arbiter strands), computes a zero or negative remaining time.
`normalizeAttachSpec` refuses a negative `Timeout` (internal/shuttleengine/attach.go:338) and replaces zero with the config's `run_timeout_min` (attach.go:359).
The first leaves an error the card maps to nothing, with the live strand never stopped; the second waits a full default timeout past D8's deadline.

### [MEDIUM] Card 14 states no bound for its new routing edges

**Location:** `_lyx/plan/14-driver-arbiter-branch.md:25`

**Issue:** The rubric's attack-surface rule asks the introducing card to state what a new routing edge lets through and what bounds it.
Card 14 names what the `resume` re-launch skips (the spent repair cap, the blocked rule) but no bound; cards 2, 3, 10, 12 and 13 each carry one.

### [MEDIUM] Eligibility is tested twice

**Location:** `_lyx/plan/10-arbiter-verb.md:52`

**Issue:** Card 9's `arbitereligibility_test.go` holds a row per eligible and refused kind for the unexported `arbiterEligibility`.
Card 10's spawn-path table again drives "each eligible stop" and "an ineligible stop" through `arbiterVerb`, its only caller.
PATTERN-test-economy tests an unexported helper through the surface that uses it and counts a redundant test as a finding.

### [LOW] Consumption test re-asserts Relaunch's effects

**Location:** `_lyx/plan/12-relaunch-consume-attempt.md:36`

**Issue:** Card 12's rows "backward target: … an existing `pause_before` kept", "no target: … status unchanged" and the admit refusals "leaving the status unchanged" pin behavior that `TestRelaunch` (card 2) already pins: "both conditions kept", "file content unchanged", "`Admit` error (file unchanged)".

### [LOW] Waiter refusal rows belong in the existing refusal test

**Location:** `_lyx/plan/03-shedverbs-arbiter-attempt-flag.md:32`

**Issue:** `TestLoop_RefusalsBeforeAnyStep` (internal/shedverbs/loop_test.go:587) is the table of waiter refusals that run no loop, asserting kind, error and no child.
The new test's nil-hook, busy and other-error rows are refusals of that kind.

### [LOW] CheckGotoTarget changes Goto's refusal precedence

**Location:** `_lyx/plan/02-shedengine-relaunch.md:18`

**Issue:** internal/shedengine/goto.go checks `findProducer` (line 66), then `StateRunning` (line 74), then `gotoAdmitted` (line 77).
One `CheckGotoTarget` call covering both target checks has to sit on one side of the running check, so one input combination changes refusal.
The card claims the messages stay byte-identical and the existing tests pass unchanged without saying where the call goes.

## Rejected

- Fidelity: every Decision D1-D13, the three added decisions and every Constraint has a carrying card; the only defect found is F2's narrowing of D6's wake condition.
  `ArbiterRole` sitting in `internal/loomengine` rather than `internal/loomcli` (D11's wording) is not a defect: loomcli still spawns the arbiter, battencli has to read the role without importing a CLI package, and `LoomDriverStrandName` in internal/loomengine/driver.go is the precedent the Shared Decision cites.
- Card order: no card uses a symbol a later card introduces; cards 7-11 use card 1's paths and card 5's records only from card 11 on, and card 10's use of `CheckGotoTarget` follows card 2.
- Granularity: cards 1, 4 and 6 each declare exported symbols used from another package (`shedrun` paths from loomcli and battencli, `ResolveArbiter` from loomcli, `PendingEscalation` from loomcli), so each is its own reviewable unit.
- Member glyphs and re-sign arrows: every `Edit` card names member glyphs for the symbols its Intent changes; no edited function changes its signature (`driverPrompt` already takes `runID`, and `Hooks`, `InnerRunDeps` and `decisionDeps` gain struct fields, which take no arrow); the file paths listed (`doc.go`, `circling.go`'s file comment, `cli.go`'s package doc) are file-level edits.
- Test economy on cards 5 and 11: card 5's tests sit at loomengine's exported surface, and card 11 adds only a row to `TestStatusCommitPathspec`.
- Verify coverage: `go build ./...`, the three `go vet` tiers, `go test ./...` and `go test -tags integration ./...` run in the template's order, and the repo's only nested module (`internal/planglyph/testdata/glyphchain`) is targeted by no card.
- Darn coverage: darn is a recipe armed through loomcli (`loomrecipe.BouncerRunSubdir` reads both recipes), so card 12's hook in `specFor` and card 14's marker in `driverPrompt` reach darn runs too.
