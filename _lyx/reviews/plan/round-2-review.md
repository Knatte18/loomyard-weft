---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/00-overview.md
    summary: "Card target lists keep omitting files the card's own change forces to change, so verify fails (test-budget.yaml, the shared Env fixtures and the batten seam test) and a doc goes stale; round 1 found the same gap in two other files"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-two-agent-round.md
    summary: "Card 7 maps a failed start to OutcomeDied with NotStarted as RunGated did, but StartGated returns only (nil, err), so the half's outcome class, identity, run dir and the start error text are unavailable and the plan says neither what is reported nor tests it"
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-two-half-resume.md
    summary: "RunnerShuttle is exported, wired from burlercli and loomcli, and card 8 gives it a nil-not-typed-nil contract, yet every planned test runs over a fake Shuttle so the real adapter has no hermetic test"
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-per-round-models.md
    summary: "Card 3 gives way-forward wording and a refusal-spec row to the LoadConfig errors only, but ResolveReview now also refuses a bad fix or later-round entry at lyx loom start with no way-forward clause and no row"
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-two-half-resume.md
    summary: "The burler producer's new failed-strand-removal exits and ErrHalfNotStopped match the trigger of the existing loom row producer re-run superseded strand not removed, but no card requires the lyx reed remove way-forward clause in their messages or extends that row"
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/02-ready-marker-path.md
    summary: "burlermarker.Path errors on a review path outside root, a new refusal for lyx burler run, whose profile paths may be absolute and outside the target today; the card neither says so nor names a way forward"
---

### [BLOCKING] Card target lists omit files the change forces to change

**Location:** `_lyx/plan/00-overview.md` (the method); the instances sit in `02-ready-marker-path.md`, `08-two-half-resume.md` and `07-two-agent-round.md`.
**Issue:** Round 1 raised two scope gaps of this kind (`template_test.go` left uncompilable by the deleted stencil embed, `docs/overview.md` not updated), and this round finds three more, so the gap is a method one: a card lists the files it edits by reading the symbols it names, and does not sweep the repository for the guards, fixtures and docs that count or enumerate what the card changes.
The instances:

1. **Test budget.**
   `cmd/lyx/testbudget_test.go` (`TestTestBudget_PackagesStayWithinBudget`) fails when a package has more top-level `Test…` functions than its row in `cmd/lyx/testdata/test-budget.yaml`, or has tests and no row.
   Card 2 creates `internal/burlermarker/marker_test.go`, and the package has no row.
   The current counts equal the rows exactly for `internal/burlerengine` (32 of 32), `internal/burlercli` (18 of 18) and `internal/shuttleengine` (140 of 140), so any added top-level function breaks the gate.
   The cards add such functions: card 1's new attach test, card 3's `models_test.go` and card 6's `remover_test.go` (both `burlerengine`), card 4's await test in `burlercli`, and card 7 and 8's per-behaviour tests in `engine_test.go`.
   The file's header says a raise lands in the commit that adds the tests, no card names it, and `go test ./...` is in the plan's `## verify:`.
2. **`Env.BurlerRemover` fixtures.**
   Card 8 adds `Env.BurlerRemover` and makes `burlerRoundEntry` require it.
   `internal/testkit/envkit/envkit.go` `FullEnv` fills `Burler` and `Shuttle` but not the remover, so `envkit_test.go`'s check that `NilSeams(FullEnv(t))` is empty fails, and so does every suite that builds the BurlerRound row from it (`internal/shedbuild`, `internal/loomrecipe`).
   `internal/shedrecipe/fixture_test.go` `newTestEnv` has the same gap, so every BurlerRound row in `entries_burler_test.go` is refused at construction.
   `internal/battencli/wire_seams_test.go` lists each Env seam batten leaves nil in `intentionallyNil` (it already lists `Burler`), so the new interface field makes `TestWire_EverySeamFilled` fail.
   `FullEnv`'s own doc comment says a seam added to the registry's requirements must be filled there.
3. **Stale doc.**
   `docs/shared-libs/stencil.md:77` says the burler round passes "the thin round orchestrator" as a value and writes "the three per-step instruction files it names".
   Card 7 replaces that with two orchestrators and four instruction files and does not list the doc.

**Fix:**
- Add `cmd/lyx/testdata/test-budget.yaml` as an `**Edit:**` target of each card that adds a top-level test function, raising that package's row by the count the card adds, and add an `internal/burlermarker` row in card 2; where a planned test can be a row of an existing table-driven test, plan it as a row.
- Add `internal/testkit/envkit/envkit.go`, `internal/shedrecipe/fixture_test.go` and `internal/battencli/wire_seams_test.go` as `**Edit:**` targets of card 8, filling `BurlerRemover` in the two fixtures and adding a `BurlerRemover` row with a loom-only reason to `intentionallyNil`.
- Add `docs/shared-libs/stencil.md` as an `**Edit:**` target of card 7 and reword its `burler` bullet for the reviewer and fixer orchestrators.
- Add one sentence to the overview's Shared Decisions: a card that changes an `Env` field, a registered stencil, a seam signature or a package's test count greps for the fixtures, guard tests, budget row and docs that enumerate it, and lists each as a target.

### [MEDIUM] A half that fails to start loses its Result under `StartGated`

**Location:** `_lyx/plan/07-two-agent-round.md`
**Issue:** The card says a start failing with `ErrNotStarted` is that half's `OutcomeDied` with `NotStarted` set and a nil error, "as `RunGated` reported it".
`RunGated` builds that Result from `start`'s own return, which carries the died-or-timeout class `abandonStartup` chose and the run's `SessionID`, `StrandGUID` and `RunDir` (`internal/shuttleengine/run.go:653-668`, `wait.go:527-548`).
`StartGated` returns `(nil, err)` and drops it (`run.go:325-333`), and the card discards the error.
So the half's identity fields are empty, its class is a guess, and the error text that names the run dir, the strand and whether the strand could not be removed never reaches the operator.
`BurlerProducer`'s double-failure message prints `SessionID` and `RunDir` (`internal/shedadapters/burler.go:459`), and D7 gives the per-half identities so an operator can inspect a failed round.
The decision record allows exactly one shuttle addition, the probe, so the plan cannot widen `StartGated`.
**Fix:** State in the card what a not-started half reports: `OutcomeDied`, `NotStarted`, empty identity, and the start error's text carried onto the round's failure message so the run dir and strand reach the producer's error.
Add a row to the existing start-failure table in `engine_test.go` that pins it.

### [MEDIUM] The real `RunnerShuttle` adapter has no hermetic test

**Location:** `_lyx/plan/08-two-half-resume.md`
**Issue:** `RunnerShuttle` is exported, built by `burlercli/wiring.go` and `loomcli/wiring.go`, and wraps a `*shuttleengine.Run` into `Handle`.
Card 8 states its contract: `ProbeGated` "returns a nil `Handle`, never a typed-nil one, on the not-found answer", and the same trap applies to `StartGated`'s error return in card 7.
Every planned test drives a fake `Shuttle` with fake handles, so the adapter's real code is reached only by the `llm`-tier smoke tests, which a typed-nil never fails.
A hermetic test is feasible: `internal/websterengine/recoverbatch_test.go` already runs a real `*shuttleengine.Runner` over `shuttlefake.Engine` and `shuttlefake.Reed`.
**Fix:** In card 8, add rows to `engine_test.go` over `RunnerShuttle(shuttleengine.NewRunner(…shuttlefake…))`: `ProbeGated` with no candidate returns an interface-nil `Handle` and `found` false, a `StartGated` error returns an interface-nil `Handle`, and a found run returns a handle whose `StrandGUID` and `RunDir` name it.

### [MEDIUM] New `ResolveReview` refusals carry no way forward and no spec row

**Location:** `_lyx/plan/03-per-round-models.md`
**Issue:** The card adds way-forward wording and a `refusal-spec.md` row beside `config: review key below 1` for the `LoadConfig` list errors only ("These load errors").
It also makes `ResolveReview` resolve every entry of both lists "so a bad later-round entry fails before round 1", and adds the `fix` key.
A registry refusal of a `fix` entry or of a later `review` entry stops `lyx loom start` before round 1, so it is a new refusal reachable through a loom verb.
Its message is `loom: ResolveReview: review role model-spec: <registry error>` today (`internal/loomengine/review.go:62-70`) and the card requires no way-forward clause, no row, and no test beyond "a bad later entry fails" (`PATTERN-refusal-way-forward`).
**Fix:** Have `ResolveReview`'s per-entry errors name the key and entry index and end with a way-forward clause, extend the row the card already adds (or add a sibling row) to cover the resolve-time refusal, and make the `review_test.go` row assert the clause.

### [MEDIUM] The producer's strand-removal failures need the existing row's way forward

**Location:** `_lyx/plan/08-two-half-resume.md`
**Issue:** Card 8 has `BurlerProducer` stop a live half through `Remover`, and returns without archiving or spawning on a failed removal, a probe error, or a `Resume` error wrapping `ErrHalfNotStopped`; card 7 returns the same sentinel from `join`.
That is a producer row re-run that finds a live strand of an earlier run and cannot remove it, which is the trigger of the loom row `producer re-run: superseded strand not removed` (`contracts/specs/refusal-spec.md:242`).
Its way forward is `run "lyx reed remove <guid>", then re-step the row`, and shuttle's own message carries that clause (`internal/shuttleengine/attach.go:192`).
The cards give these errors no wording, so a resumed Burler row can halt a loom run with a message that names no way forward, and the row's coverage of BurlerRound is not stated.
**Fix:** Require card 7's `ErrHalfNotStopped` error and card 8's removal-failure returns to end with `way forward: run "lyx reed remove <guid>", then re-step the row` (naming the stuck half's guid), extend that row's trigger to the BurlerRound row in `refusal-spec.md` in the same card, and have the existing failed-removal rows assert the clause.

### [LOW] `burlermarker.Path` refuses a review path outside `root`

**Location:** `_lyx/plan/02-ready-marker-path.md`
**Issue:** The card errors, naming both paths, when `reviewPath` is outside `root`.
In standalone geometry `root` is the target, and `Profile.validate` accepts an absolute `review-path` anywhere (`resolvePath` keeps it verbatim, `internal/burlerengine/profile.go:203-211`), so card 7's `lyx burler run` now refuses a profile it ran before.
D5 defines the marker only for a path under the target, and the card states the refusal in its rule but names no way forward and card 7's `Long` help does not mention the constraint.
**Fix:** Say in card 7 that `lyx burler run` now requires `review-path` under the target, put that in the verb's `Long` help, and have the error end with the way forward (move the review path under the target); or derive the marker for an outside path some other defined way.
