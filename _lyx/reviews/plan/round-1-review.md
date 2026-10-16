---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-stall-freshness.md:21
    summary: Card 3 creates twin rig.probeUntilDone helpers in internal/creelengine_test and internal/creelcli, duplicating test support across two packages (PATTERN-testkit) while the creelfake kit's Clock.AutoAdvance already steps a fake clock timer by timer until a done channel closes
    fix: In _lyx/plan/03-stall-freshness.md drop both Create lines, `plan:internal/creelengine_test#rig.probeUntilDone` (line 21) and `plan:internal/creelcli#rig.probeUntilDone` (line 22), and add `internal/testkit/creelfake#Clock.AutoAdvance` under **Uses:**. In the Intent sentence at line 48 replace "`rig.probeUntilDone` and its `internal/creelcli` twin advance the daemon clock by each new probe wait until the call returns" with "the rows run the daemon clock's `creelfake.Clock.AutoAdvance` until the call returns, so a probe reads at each of its waits". If probe-only stepping turns out to be needed, the alternative is one new method on `internal/testkit/creelfake#Clock` that both packages call, never a per-package copy. The combined overlay with F2 passes validate-plan --overlay with no new finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-startup-gate-seam.md:36
    summary: Card 10 adds a NoTrustInputs field to fakeEngine for a TrustDismissSequence that answers no inputs, which the existing undismissableEngine in internal/shuttleengine/startup_test.go already provides for that exact purpose
    fix: In _lyx/plan/10-startup-gate-seam.md drop `internal/shuttleengine#fakeEngine` (line 5) and `internal/shuttleengine#fakeEngine.TrustDismissSequence` (line 6) from **Edit:**, add `internal/shuttleengine#undismissableEngine` under **Uses:**, delete the paragraph at line 36 that adds `NoTrustInputs`, rewrite the test row at line 41 as "a gate capture over the existing `undismissableEngine`, whose sequence is empty, sends no key and reports false", and drop "one fake field" from the ImpactSummary. The combined overlay with F1 passes validate-plan --overlay with no new finding.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-stall-freshness.md:49
    summary: Card 3's new TestCreelCLI row for an inject --wait after a draft cleared under a hold pins nothing the engine row StallClearedUnderAHoldIsNotSeededIntoALaterWaiter and the existing CLI row on printed stalls do not already pin
    fix: In _lyx/plan/03-stall-freshness.md delete the paragraph at line 49 that adds the TestCreelCLI row "an inject --wait started after a draft cleared under a hold prints no stall line and ends injected". The no-seed behavior is pinned by the new TestScheduler_Injects row "StallClearedUnderAHoldIsNotSeededIntoALaterWaiter", and the CLI printing a progress note as an `is stalled` stderr line, or nothing for no note, is pinned by the existing TestCreelCLI row "a waiting inject hears the stall on stderr and an expired answer carries it". Keep `internal/creelcli#TestCreelCLI` in **Edit:**, since that existing row still changes how it steps the clock. No target changes.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/06-sweep-confirm-timing.md:40
    summary: Card 6 shortens the two-miss guard from at least 30 s to at least 10 s and states its bound, but its Intent never says what the shorter guard now lets through
    fix: In _lyx/plan/06-sweep-confirm-timing.md, append to the Bound sentence at line 40 the sentence - The shorter guard now lets a strand that is absent from reed's persisted state for 10 seconds or more, for example while it is removed and re-added under the same name, have its held mail returned along its path, where before it had to be absent for at least 30 seconds. No target changes.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/11-llm-smoke-strand.md:18
    summary: Card 11's pre-idle poll says it runs until the pane shows its input box or a startup gate, but names no reading for the input-box exit and does not say whether an accepted gate ends the poll
    fix: "In _lyx/plan/11-llm-smoke-strand.md, at line 18, name the poll's exit conditions: each tick calls `Runner.AcceptStartupGate` on the strand, and the poll ends at the first tick where that call reports true or where `hubDelivery.Idle`'s answer carries no `no-input-box` stall; at `smokePaneWindow` the test fails and prints the strand's last pane capture. No target changes, since `internal/creelcli#hubDelivery.Idle` is already in **Uses:**."
---

### [BLOCKING] Twin `rig.probeUntilDone` helpers duplicate test support across packages

**Location:** `_lyx/plan/03-stall-freshness.md:21`

**Issue:** Card 3 creates `rig.probeUntilDone` in `internal/creelengine_test` and a second copy in `internal/creelcli`, and its Intent calls the second one a twin.
Both advance a `creelfake.Clock` by each new probe wait until a call returns.
`PATTERN-testkit` says test support used by two or more packages is shared through one kit under `internal/testkit/<kit>/` and is never duplicated per package.
Both packages' rigs already drive the kit clock `internal/testkit/creelfake`.
That kit's `Clock.AutoAdvance(done)` (`internal/testkit/creelfake/creelfake.go:295`) moves the clock to each pending timer in turn until `done` closes.
`server_test.go` already uses it for calls that retry on the clock.
Probe waits are at most 8 seconds and come before the 30-second sweep timer, so stepping timer by timer has a probe read the stall at each wait, which is what the three rows that today jump 61 seconds need.
The existing `sweepStep`/`sweepLast`/`isProbeWait` twins are already in the tree and are not this plan's subject.
Card 3 adds a new duplicate.

### [MEDIUM] `NoTrustInputs` on `fakeEngine` duplicates `undismissableEngine`

**Location:** `_lyx/plan/10-startup-gate-seam.md:36`

**Issue:** `internal/shuttleengine/startup_test.go:56-67` already defines `undismissableEngine`, an embedded `*fakeEngine` whose `TrustDismissSequence` returns no inputs.
Its doc comment calls it "the shape an implementation that cannot locate the gate's accepting option takes".
Card 10 instead edits the package's shared `fakeEngine` and its `TrustDismissSequence` to add a second switch for the same behavior.
The row that uses it asserts only that no key is sent and that the call reports false, so it does not need the recorded capture that only the new field would add.

### [MEDIUM] New CLI row repeats the engine row and the existing CLI stall row

**Location:** `_lyx/plan/03-stall-freshness.md:49`

**Issue:** Under `PATTERN-test-economy`, a new test must pin public-surface behavior that no test covers yet.
The engine row "StallClearedUnderAHoldIsNotSeededIntoALaterWaiter" pins that a waiter started after a draft was cleared under a hold gets no seed and ends injected.
The existing `TestCreelCLI` row "a waiting inject hears the stall on stderr and an expired answer carries it" (`internal/creelcli/cli_test.go:423`, `:432`) pins how the CLI turns progress notes into `is stalled` stderr lines.
The new integration-tier row needs a hold, a role identity and stepping past the freshness bound, and it pins only the combination of those two behaviors.
The decision record's test constraint says a test extends an existing table where one already covers the behavior.

### [LOW] Card 6's Intent states the bound of the shorter guard but not what it lets through

**Location:** `_lyx/plan/06-sweep-confirm-timing.md:40`

**Issue:** The attack-surface rule asks the card that downgrades a guard to say in its Intent both what the change can now let through and what bounds it.
Card 6 states the bound: two successful listings at least 10 seconds apart.
It never says what the change admits.
The decision record's Open risks names it: a strand absent from reed's persisted state for more than 10 seconds while it is re-added under the same name loses its held mail.

### [LOW] Card 11's gate poll leaves its exit conditions unnamed

**Location:** `_lyx/plan/11-llm-smoke-strand.md:18`

**Issue:** D9 says the test waits for the pane to show either its input box or a startup gate, and accepts the gate when it shows one.
The card says only that the poll calls `Runner.AcceptStartupGate` each tick "until the pane shows its input box or a startup gate".
It does not say which reading tells the box is showing, since `internal/creelcli` may not import `claudeengine`.
It also does not say whether a `true` from the seam ends the poll.
`hubDelivery.Idle` is in the card's **Uses:**, but nothing in the Intent ties it to the poll.

## Rejected

- Verify coverage: no defect.
  `internal/creelcli` holds `tmux` and `llm` files (`smoke_tmux_test.go`, `smoke_llm_test.go`, `smoke_helpers_test.go`), and `## verify:` only runs `go vet` over them.
  `loom-plan-spec.md`'s verify model says a plan's `## verify:` compiles the `tmux` tier and never runs it (Publish runs it through `publish_verify`), and compiles but never runs `llm`.
  The `integration` files of `internal/creelcli` (`cli_test.go`, `delivery_test.go`) are run by `go test -tags integration ./...`.
  `internal/creelengine` and `internal/shuttleengine` hold no tag besides the platform pair `peer_other.go`.
  The hub's verify-tier list is `none`, so only the coverage rule applies, and it holds.
- Card order: card 11 uses `Runner.AcceptStartupGate`, which card 10 creates earlier, and card 8 reads `IdleProbe.NoInputBox`, which card 7 adds earlier.
  Cards 3 and 4 both edit `TestDaemon_Protocol`, cards 4 and 5 both edit `TestDaemon_Lifecycle`, and cards 1 and 5 both edit `TestJournal_Fold`, each pair in order.
  No card depends on a later one.
- Card 3 against card 8: `worker.probe` ages injects before it reads the pane, so after card 3 the two-minute jump in "InjectStalledForItsWholeTTLStillExpiresWithOneStalledRecord" would expire with a stale stall.
  That row asserts only the outcome kind and the stalled reasons, not the outcome's stall, so it passes after card 3, and card 8 later adds the stall assertion and its probe-by-probe stepping.
- Card 9 against "InjectPostedMidWaitIsChargedFromItsPostingNotFromThePreviousProbe": traced through `worker.run` and `probe`, the card's claim holds.
  The probe at the 59-second mark charges 59 seconds, the wait after it is capped at 1 second, and the next tick expires the inject.
- Fidelity: every Decision is carried.
  D1 by card 1, D2 by card 2, D3 by card 3 (the per-waiter last-told stall gives once per change, so A, B, A is told three times), D4 by card 4, D5 by card 5 (one `carried` entry per carry, pinned by the second rotation), D6 by card 6 (every pending-confirm rule, the 50-second ceiling and the failed-listing rule), D7 by cards 7 and 8, D8 by card 9, and D9 by cards 10 and 11.
  Every Constraint is carried as well, and so is every doc and suite line the Scope names (C4 by card 2, C6 by card 9, C8 by card 3, C10 by card 1, C11 by card 6).
- Attack surface on cards 4 and 10: both state what the change lets through and what bounds it.
- Reachability: each prescribed refusal or guard branch has a named input and a named test, and no upstream check rejects the input.
  For card 2 that is `failingWriter` over both reads and over the two `runVerb` envelopes.
  For card 4 it is an already-fired non-race exit with no daemon serving.
  For card 10 it is `liveStrandStatus(false)` for the dead strand and a run record that `FindRun` cannot read, which `interruptEngine` returns as a non-`ErrNoRunRecord` error.
- Edit glyph shape: `runCreelInto`, `NewClient`, `spawnDaemon` and `sweepOnce` change their signatures and carry the re-sign arrow.
  `clientSeam` and `Waiter` are struct edits and take no arrow.
  `rig.client` keeps its `spawn func() error` parameter.
  No `Edit` card names symbols while listing only non-test Go file paths.
- Card 10's `//lyx:guard`: `TestProviderSeamImportRule` already reads files across the module and carries only `//testtiming:keep`, so adding the guard marker is in scope.
