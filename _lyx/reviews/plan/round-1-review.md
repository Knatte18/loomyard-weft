---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/05-shedrecipe-plan-write-forked-gate.md:20
    summary: Card 5's wiring subtest cannot tell NewLensReviewPlanGate from NewPlanGate, so no test pins that the Plan-Write row's Forked closure carries the lens-review check
    fix: "In card 5, add two Create targets, `plan:internal/shedrecipe#noFindingsPlanIndex` -> `type noFindingsPlanIndex struct` and `plan:internal/shedrecipe#noFindingsPlanIndex.ValidateFormat` -> `func (noFindingsPlanIndex) ValidateFormat(*planparser.Plan, string, []planparser.Card) ([]planindex.Finding, error)`, a test index in fixture_test.go that embeds planindex.Index and answers no findings from ValidateFormat. Rewrite the subtest PlanWriteSetsForkedOnItsPlanEntry: set env.PlanIndex to noFindingsPlanIndex{}, seed a minimal parseable plan (00-overview.md plus one card file) under planparser.PlanDir(env.AnchorPath) with no lens-review.md, and assert that Gate() passes while Forked() fails with findings naming lens-review-missing. This combined target change was checked with lyx loom validate-plan --overlay and adds no blocking finding."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-shuttle-turn-in-flight-veto.md:19
    summary: Card 1 changes clearableBoxEngine.ParseSessionSignals and adds TurnStartProgress methods to two fakes, but its target list names only the fake types
    fix: "In card 1's Edit group, add `internal/shuttleengine#clearableBoxEngine.ParseSessionSignals` after `internal/shuttleengine#clearableBoxEngine`. In its Create group, after the TurnProgressReader line, add `plan:internal/shuttleengine#sessionFakeEngine.TurnStartProgress` -> `func (e *sessionFakeEngine) TurnStartProgress(SessionSignal) TurnProgress` and `plan:internal/shuttleengine#clearableBoxEngine.TurnStartProgress` -> `func (e *clearableBoxEngine) TurnStartProgress(SessionSignal) TurnProgress`. This combined target change was checked with lyx loom validate-plan --overlay and adds no blocking finding."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-claudeengine-turn-progress.md:47
    summary: Card 2 has every transcript fixture written by hand from a described shape, while the decision record's acceptance criterion wants fixtures cut from the bench transcripts
    fix: "In card 2's fixture paragraph, say that each fixture line whose shape appears in the bench transcripts (a string prompt, a task notification, a tool_result user entry, a tool_use assistant line, a thinking line with stop_reason tool_use, a thinking line with stop_reason end_turn, an end_turn text line, a system turn_duration entry) is copied verbatim from ftp-r-1 or ftp-s-1 under the hub prime's .scratch/twill-fork-timing-plan/after/<run>/transcripts/, so all of the line's real keys stay. Only shapes the bench lacks (the interrupt marker, the API-error entry, the null and max_tokens stop reasons, the unknown type, the malformed and unterminated lines) are hand-built. The file names and expected readings are unchanged."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loomshed-lens-review-gate.md:20
    summary: Card 4's finding detail restates the lens-review record format (one line per finding with its lens and disposition) instead of only pointing at the directive, as D3 words it
    fix: "In card 4's Intent, end the detail sentence at 'says to record the lens review there as the lens review directive describes', and drop the clause ', one line per finding with its lens and whether it was fixed or rejected and why'. The format then lives only in the twill directive's step 7."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/00-overview.md:45
    summary: Card 6 targets contracts/specs/refusal-spec.md, which sits in the Go package contracts/specs, but no verify command runs that package's tests
    fix: "In 00-overview.md's verify section, add ./contracts/specs right after ./contracts/stencils in each of the three go vet lines and the two go test lines. This change was checked with lyx loom validate-plan --overlay and adds no blocking finding."
---

### [LOW] Card 5's wiring test does not tell the lens-review closure from the plain plan gate

**Location:** `_lyx/plan/05-shedrecipe-plan-write-forked-gate.md:20`

**Issue:** The new subtest runs `Forked()` over `newTestEnv`'s empty anchor (`internal/shedrecipe/fixture_test.go`), so the parse fails at "plan overview not found" before the lens-review check is reached.
A `Forked` wired by mistake to `loomshed.NewPlanGate` would pass that subtest too, since `TestPlanWriteEntry_GateConfig`'s existing subtest asserts the same failure for `Gate`.
Card 4 pins the closure's verdict and card 3 pins its selection, but nothing pins the Plan-Write row wiring that joins them, and that wiring is the core of D3 and of the acceptance criterion "A twill Plan-Write gate over a valid plan with no `lens-review.md` ... does not pass".
`placeholderIndex` panics on any call, but `ValidatePlan` (`internal/loomshed/gates.go:172`) calls only `index.ValidateFormat` when no webster record exists.
So a stub that answers no findings from that one method lets a parseable plan reach the lens-review check, and the two closures then give different verdicts.
`envkit`'s `noFindingsIndex` does the same job, but it is unexported in a kit that builds shedrecipe's Env, so shedrecipe's in-package test cannot reuse it.

### [LOW] Card 1's target list omits a changed fake method and two new fake methods

**Location:** `_lyx/plan/01-shuttle-turn-in-flight-veto.md:19`

**Issue:** The Intent says `clearableBoxEngine.ParseSessionSignals` (in `internal/shuttleengine/run_test.go`) also parses `SESSIONSTART` lines, and that `sessionFakeEngine` and `clearableBoxEngine` each gain a `TurnStartProgress` method.
The target list names only the types `sessionFakeEngine` and `clearableBoxEngine`.
A method is its own member glyph, separate from its receiver's type, so the changed method and the two new ones are missing from the card's declared footprint.

### [LOW] Card 2's fixtures are not cut from the bench transcripts

**Location:** `_lyx/plan/02-claudeengine-turn-progress.md:47`

**Issue:** The decision record's acceptance criterion reads "`claudeengine`'s reading over transcript fixtures cut from the bench transcripts", and its notes give the transcripts' location.
Card 2 instead describes each line as a hand-built object carrying `type`, `isSidechain`, `timestamp`, `uuid` and `message`, which drops every real key a bench line carries.
That weakens the tolerant-decoding claim (`PATTERN-tolerant-records`), which the fixtures exist to pin.
The transcripts exist: `loomyard/.scratch/twill-fork-timing-plan/after/ftp-r-1/transcripts/464047e2-….jsonl` holds every shape the card lists for the in-flight and over readings.
Those include thinking lines with `stop_reason: "tool_use"` and `"end_turn"`, `<task-notification>\n<task-id>` user entries and `system:turn_duration` entries.
The card's described shapes match those lines, so this departs from the answer key but not from the real format.

### [LOW] Card 4's finding detail duplicates the directive's record format

**Location:** `_lyx/plan/04-loomshed-lens-review-gate.md:20`

**Issue:** D3 says the finding "says to record the lens review there as the directive describes", which is a pointer.
Card 4 adds the format itself ("one line per finding with its lens and whether it was fixed or rejected and why"), which copies step 7 of `contracts/stencils/twill/twill-directive.md`.
A Go string then repeats a stencil's contract, and the two drift apart silently when the directive changes.
`PATTERN-producer-pointer-rule` names instruction files, not gate re-prompts, so this is a consistency defect against D3's wording, not a PATTERN violation.

### [LOW] Verify does not run contracts/specs, which card 6 targets

**Location:** `_lyx/plan/00-overview.md:45`

**Issue:** Card 6 edits `//contracts/specs/refusal-spec.md`, and `contracts/specs` is a Go package with its own tests (`specs.go`, `specs_test.go`).
The verify section runs `./contracts/stencils` but not `./contracts/specs`.
The plan template requires the section to cover every package a card targets.
That package's tests do not read `refusal-spec.md`, which is not embedded, so the gap can hide no failure today, and the finding is LOW.
`pattern/` and `docs/` hold no Go files and need no verify entry.

## Rejected

- Card order (focus item 2): card 3 adds `RunState.ForkSubagents` and `GateEntry.Forked`, card 4 creates `ValidatePlanLensReview` and `NewLensReviewPlanGate`, card 5 uses both, and card 6 uses card 4's function, so no card uses a symbol a later card creates. Card 3's ImpactSummary names card 5 only in prose. Card 7 names card 6's command only as text.
- Test economy (focus item 3): card 1 adds only rows to `TestSend_WaitsForIdleSession`, `TestGate_WaitsForIdleWriter` and `TestRunner_SessionIdle_ReturnsScriptedClassification`. `readinessEngine` embeds `sessionFakeEngine` through `idleEngine`, and `tuiEngine` embeds `*clearableBoxEngine`, so the `progress` fields reach every rig. Cards 4, 5 and 6 add subtests or rows only. Card 2's `TestTurnStartProgress` is the one new top-level test, for a method no test covers, and its budget bump from 110 to 111 matches the count today, which is 111 including TestMain.
- Attack surface and reachability (focus item 4): cards 1, 3, 4, 5 and 6 each state a bound in Intent. Card 6's three flag refusals have named inputs, named test rows, way-forward clauses and refusal-spec rows. Card 4's other-read-error branch is a raw I/O error that the gate returns, not a refusal.
- Re-sign arrows (focus item 5): `unmatchedTurnStart` and `turnStartHold.holds` carry them. No other listed member changes its signature.
- Fidelity (focus item 1): the SessionIdle busy reading is the "an in-flight turn start stays held past the idle override" row of `TestRunner_SessionIdle_ReturnsScriptedClassification`. It reads "a turn is running" because the veto keeps the fold's `SessionBusy` state out of the released-turn branch of `sessionReadiness` (`internal/shuttleengine/readiness.go:110`). The session-start exception is the `restarted` result and its two send rows and one gate row. Both attach cases are card 3's two `TestAttach_GateThreading` rows. Every item on the docs list has a card: shuttleengine doc.go in cards 1 and 3, the claudeengine doc in card 2, the plan.go file doc and the twill paragraph of docs/overview.md in card 7, and the validate-plan sentences and the rewritten Pairs row in card 6. The anchor sentences each card quotes exist in the files.
- The SessionIdle unknown-state path: with unproven liveness, `Runner.SessionIdle` (`internal/shuttleengine/session.go:138`) falls back to the pane and never consults the hold, so it reports idle at once, as it does today. D1 claims busy only where today it reports idle after 10 s, which is the known-state path, so the plan matches the answer key.
- Card 3 and older records: the attach callers (`shedadapters/singlellm.go:147`, `bouncer.go`, `frictionengine/reflect.go`) read no `ForkAudit`. Burler's `auditClusterRound` reads one, but burler never attaches, so moving the audit source to the record changes no consumer's answer beyond what D3 states.
