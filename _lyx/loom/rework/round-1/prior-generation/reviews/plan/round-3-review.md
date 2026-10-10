---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-discussion-seats-entry.md
    summary: TestDiscussionSeatsEntry_Call scripts done chair results with a gated table but no GateOutcome, so card 6's fake evaluates the row's gate, which fails the commit row and opens a parent-review round in the resume row
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-discussion-write-row-selection.md
    summary: the pre-build engine substitution puts a gated DiscussionSeats row past the gate-quiescence guard, which reads only the parsed recipe file and lists only the four writer engines, and card 8's Intent states no bound for that
---

### [MEDIUM] Card 7's Call test conflicts with the gate-once fake from card 6

**Location:** `_lyx/plan/07-discussion-seats-entry.md`, the `TestDiscussionSeatsEntry_Call` paragraph; `_lyx/plan/06-seat-runner-gate-once.md`.

**Issue:** From card 6 on, `shedfake.SeatRunner.Run` and `Resume` evaluate `table.Gate` once whenever the gate is non-empty, the chair is done and the answered chair result carries no `GateOutcome`.
Card 7's entry wraps the source so the evaluated table carries the row's resolved gate, and card 7's test scripts two rows against that fake without saying the chair result carries a `GateOutcome`:

- "a done chair asserts ... the table the runner saw carries the row's resolved gate by name and budget, the pointer is the chair's first output and the commit fired once".
  With a gate such as `discussion` over `newTestEnv`, whose support log path is never created (see `TestDiscussionWriteEntry_GateConfig` in `internal/shedrecipe/entries_discussionwrite_test.go`, where the same gate reports "support log does not exist"), the fake stamps a failed outcome.
  The decorator then reports Stuck, and the commit never fires.
- "a live chair scripted through `ProbeFn` ... and `ResumeFn` with a done result opens none".
  The fake now evaluates the `parent-review` gate on that resumed done chair.
  That gate opens a round, as the existing `AttachDoesNotBeginRound` subtest of `TestDiscussionWriteEntry_ParentReviewBeginRound` notes for the fake shuttle: it scripts a Died attach for exactly that reason.
  So "opens none" fails because of the fake, not because of the preparation under test.

Card 6's ImpactSummary says `TestMultiLLMEntry_ComposedTable` is the fake's only consumer with a gated table, which was true when it was written, but card 7 adds two more.
As planned, `go test ./internal/shedrecipe` would fail after card 7.

**Fix:** In card 7, state that the done-chair row scripts its chair result with a passing `GateOutcome`, as card 6 does for `TestMultiLLMEntry_ComposedTable`, or that it builds the entry with no `gates` key.
Then assert the gate's name and budget on the table the runner saw.
State that the resume row's `ResumeFn` result also carries a `GateOutcome`, or uses a non-done outcome, as the single-agent attach subtest does.
Optionally, widen card 6's ImpactSummary sentence to cover the later consumers.

### [MEDIUM] The Discussion-Write row substitution bypasses the gate-quiescence guard with no stated bound

**Location:** `_lyx/plan/08-discussion-write-row-selection.md`, the `applyDiscussionProducer` and `New` sentences; `internal/loomrecipe/gatequiescence_test.go`.

**Issue:** The decision record's Constraints say the chair is gated and never forks, and D1's rationale kept the engine name so that "the gate-quiescence writer-engine list" stays as it is.
The parent's later decision gives the seat-table producer its own engine name, `DiscussionSeats`, so that premise no longer holds.
`gatedRowFanViolations` admits a gated row only when its engine is `BurlerRound` or one of `DiscussionWrite`, `PlanWrite`, `Describe` or `PRRework`.
`TestNoGatedRowAuthorizesForkSubagents` runs it only over `shedbuild.Parse(recipes.LoomRecipe)`.
Card 8 rewrites the row's engine to `DiscussionSeats` inside `New`, after the parse, so the built gated row never passes through that guard.
If it did, the guard would flag it as neither a writer nor `BurlerRound`.
That is a routing edge that skips a guard.
Card 8's Intent does not say what the substituted row now escapes, or what keeps the chair fork-free: the seat engine's spec, which never sets `ForkSubagents`.
The shared decision `two-producers-selected-by-config` claims the coverage guard stays honest, but it says nothing about this guard.

**Fix:** In card 8, put `internal/loomrecipe#gatedRowFanViolations` and `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents` under `Edit:`.
Add `DiscussionSeats` to the writer-engine case and its message.
Add a row that runs the guard over the recipe after `applyDiscussionProducer(recipe, true)`.
Add one Intent sentence: the substituted row is gated on the chair, a seat spec never sets `ForkSubagents`, and the guard covers the substituted recipe.
Alternatively, state the bound in the Intent and explain why the guard need not cover it.

## Focus verdicts

- Env seams and `TestWire_EverySeamFilled`: not a defect.
  Card 7 adds the func-typed `Env.DiscussionTable`.
  In the same card, it fills that field in `loomcli`'s `wire`, lists it in `battencli`'s `intentionallyNil` and fills it in `envkit.FullEnv`.
  `envkit.NilSeams` is called only from the loomcli and battencli wire tests and envkit's own tests.
  Card 8's `Env.DiscussionSeats` is a `bool`, which `NilSeams` does not report.
- Pinned results: card 8 lists `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` and the coverage-guard allowlist for the change to `RecipeEngines`.
  `RecipeEngines` has no other consumer.
  Card 7 lists `TestNames`, `minimalEntries`, `newTestEnv` and `FullEnv` for the new registry entry.
  `TestBuild_EveryRegisteredEngineBuilds` builds `DiscussionSeats` with no config, as it builds `DiscussionWrite`.
  The one gap is the fake-gate interaction in F1.
- The acceptance criterion that the discussion gate and the parent-review brief read only the two discussion files, with an advisor notes file beside them, is pinned.
  For the gate, card 7 adds a row to `TestNewDiscussionGate` that plants a failing `advisor-1.md` beside valid files.
  For the brief, card 5 adds an assertion to `TestParentReviewBrief_NamesPathsAndSubmitLines`.
  `NewDiscussionGate` takes the two paths only, and `ParentReviewBrief` fills from the two paths only, so both rows test the real surface.
