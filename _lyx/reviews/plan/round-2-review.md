---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-webster-sites-slotted.md
    summary: RecordBatch derives the card-verify wait directory inside websterengine via gateslot.WaitDir(deps.Geom.AnchorRoot) instead of being told it, breaking PATTERN-told-geometry and Geometry's never-derives contract
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/22-webster-review-failure.md
    summary: Card 22 introduces the record-driven widening of the Webster-Review range but its Bound omits what a deleted or edited record lets through there (D11's Bound), which only card 21 states for the gate
  - id: F3
    severity: LOW
    class: consistency
    location: internal/websterengine/doc.go:70
    summary: The doc still says a fork returns FAILED only on a non-zero build/unit gate or card verify, which the batch gate and the slot-busy exception make false, and no card rewrites that sentence
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/11-status-gate-waits.md
    summary: gateWaiting names only the oldest live wait record, and a live verify marker hides every gate wait, where D6 has status report each waiting gate
---

### [BLOCKING] Card 6 derives the wait directory inside webster's engine

**Location:** `_lyx/plan/06-webster-sites-slotted.md`, the sentence starting "`RecordBatch` passes `deps.Geom.GateSlots` and `gateslot.WaitDir(deps.Geom.AnchorRoot)`".

**Issue:** `internal/websterengine` is a bound package under `PATTERN-told-geometry` (listed in `pattern/PATTERN-told-geometry.md`): an engine is handed the absolute paths it works on and derives none.
`websterengine.Geometry`'s own doc comment (`internal/websterengine/geometry.go:21`) says it is "the set of paths webster is told, once, at construction, and never derives itself".
Its established form for a never-tracked per-worktree directory is a told field filled by the geometry constructors, such as `VerifyDir`, which `hubgeom.WebsterGeometry` sets to `verifytree.Dir(anchorPath)` (`internal/hubgeom/webstergeom.go:37`).
Card 6 instead has `RecordBatch`, engine code, join the wait directory from `AnchorRoot` through `gateslot.WaitDir`.
The other wait-directory derivations in the plan sit in CLI layers (`loomcli`, `webstercli`, `gatecli`), so this is the one site that breaks the constraint.

**Fix:** Card 5 adds a told field to `websterengine.Geometry` (for example `GateWaitDir string`, documented as the per-worktree directory a slotted card-verify rerun writes its wait record into), and `hubgeom.WebsterGeometry` fills it with `gateslot.WaitDir(anchorPath)`.
Card 6's `RecordBatch` passes `deps.Geom.GateWaitDir`, and `gateslot#WaitDir` leaves card 6's `Uses:`.
Card 5 already lists `internal/websterengine#Geometry` and `internal/hubgeom#WebsterGeometry` under `Edit:` and card 6 already uses `Geometry`, so no target moves.
Optionally, card 9's `wireHub`/`wireStandalone` can then read the same field instead of joining again.

### [MEDIUM] Card 22's Bound leaves out what a tampered record does to the review

**Location:** `_lyx/plan/22-webster-review-failure.md`, the `Bound:` line.

**Issue:** Card 22 adds a routing edge: a file in the pair's own `.lyx/` now widens the Webster-Review range and forces BLOCKING findings.
Its Bound says only that "the rendered record is the only widening; with no record the range rules stay as they are".
D11's Bound states that deleting or editing the record "can skip, narrow or redirect the round pass and D12's review widening", and that after the shape checks what still passes is "a well-formed test name that never failed, which forces a BLOCKING review finding for it, and another real merge commit of the worktree, which points the reviewer at that commit's diff".
Card 21 carries that bound for the gate's command alone, and says nothing about the reviewer.
Under the attack-surface question, the card that introduces the edge states what it lets through and what bounds it, so card 22 needs its own statement.

**Fix:** Extend card 22's Bound: deleting the record skips the widening, and editing it can, after `checkedPublishFailure`'s shape checks, only add a well-formed test name that never failed (forcing a BLOCKING finding for it) or point the reviewer at another real merge commit's diff; no shell text or free prose reaches the prompt, and Publish reruns both verifies in full regardless.

### [LOW] websterengine's doc keeps a stale FAILED condition

**Location:** `internal/websterengine/doc.go:70`, "a fork returns FAILED only on a non-zero build/unit gate or a non-zero per-card `verify:`".

**Issue:** After card 15, a failed batch gate also makes the report `FAILED` (D2), and a slot-busy exit from any gate step or Verify line never does.
The sentence's "only on" list no longer holds.
Card 15 rewrites the sentence that follows it ("there is no batch-wide verify distinct from its cards' own gates", line 77) and card 14 rewrites the card-gate sentence on line 80, but no card touches line 70.
This is the paragraph focus item 3 asked about: the other two packages are covered, since card 21 rewrites `internal/loomshed/doc.go:22` ("The round compiles the `tmux` and `llm` tiers and never runs them") and names the new `Packages` field in `internal/impactset/doc.go`.

**Fix:** Card 15, which already lists `internal/websterengine/doc.go` under `Edit:`, also restates the FAILED condition: a failed card gate, card `**Verify:**` or batch gate, never deviation alone and never a slot-busy exit.

### [LOW] Status names only one waiting gate

**Location:** `_lyx/plan/11-status-gate-waits.md`, "`gateWaiting` reports the worktree's live wait records … naming the oldest wait's site", and the chain order "verify, gate wait, shuttle, then parent review".

**Issue:** D6 says `lyx loom status` reports "each waiting gate (site or command) with its wait start".
Card 11 renders only the oldest live wait record.
Because `verifyWaiting` returns as soon as a marker is live, a live verify marker also hides every gate wait record.
Within one worktree, waits rarely overlap, since webster's batches, its card-verify rerun and its verify gate run one after another, so the practical gap is small.
It still departs from the decision's wording.

**Fix:** `gateWaiting` renders every live wait record (site and elapsed each, holders once), and the verify note appends the gate waits when both are live, or the card's Intent states why one wait per worktree is all that can occur.

## Focus verdicts

**Slot waits in status.**
Not a defect, apart from F4.
Every hub-mode verify site builds its `verifytree.Paths` over `verifytree.Dir(<anchor>)`: webster's gate through `hubgeom.WebsterGeometry`'s `VerifyDir`, Publish and Finalize through `landingDeps`, and the round gate through `loomcli/wiring.go`.
`reviewWaitingFor` reads the marker at that same path, so card 4's waiting marker reaches card 11's `verifyWaiting`.
Card 6's card-verify rerun writes its wait record into `gateslot.WaitDir(AnchorRoot)`, and card 9's plan-gate load writes into `gateslot.WaitDir(location.AnchorPath())`, both at the gate site (`wire` sets `env.PlanIndex` from `websterGeom.Index`) and at `validate-plan` (`wireLightweight`).
Card 11's `gateWaiting` reads `gateslot.WaitDir(loc.AnchorPath())`, and hub-mode `AnchorRoot` is `anchorPath` (`internal/hubgeom/webstergeom.go:27`), so they are the same directory.
The holders clause names the other worktree's Publish through `Pool.Holders`, so a gate stalled behind a Publish shows as waiting, with the Publish named, in the stalled run's `lyx loom status`.
F1 concerns where card 6 derives that directory, not whether status reads it.
`battencli/wire.go:483` reads the marker's liveness, not its state, so a waiting marker reading live does not breach D6's display-only Bound.

**Inheritance routing edge.**
Not a defect.
Card 10's Intent states the bound: a forged `gateslot.InheritEnv` naming a held slot runs inside that holder's slot without acquiring; card 12's deny refuses an agent command naming the variable; an operator shell or a script hiding the name is outside the bound.
Card 1's `Inherited` returns false for a path outside the slot directory or a free slot, so a forged value cannot reach past a slot that some holder already has.

**Docs in the same commit.**
Covered for `internal/loomshed` (card 21) and `internal/impactset` (card 21).
`internal/websterengine` is covered for the card-gate and batch-wide-verify sentences (cards 14 and 15), with one stale sentence left (F3).
