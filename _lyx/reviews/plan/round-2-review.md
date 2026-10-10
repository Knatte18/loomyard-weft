---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-shedrecipe-env-seats.md
    summary: Card 10 adds two nil-able Env seams but omits envkit.FullEnv and battencli's intentionallyNil, so TestFullEnv_FillsEverySeam and battencli's TestWire_EverySeamFilled fail under the card's own gate
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/11-shedrecipe-multillm-entry.md
    summary: Card 11 registers MultiLLM but omits shedbuild's TestBuild_EveryRegisteredEngineBuilds, which builds every registered engine from engineMinimalConfig and fails on a MultiLLM row with no config
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-stencil-include-form.md
    summary: Cards 2 and 3 plan the include action as a bare template call with no dot, which executes the block with nil data, so every block marker fails at execution under missingkey=error
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/09-shedadapters-multillm.md
    summary: The N = 1 equivalence is pinned only through the fake seat runner, so no test runs seatengine itself over a one-chair table as D11 asks
---

### [BLOCKING] Card 10 misses two seam-completeness tests that read every Env seam

**Location:** `_lyx/plan/10-shedrecipe-env-seats.md`

**Issue:** Card 10 adds `Env.Seats` (an interface) and `Env.Models` (`modelspec.Registry`, a `map[string]Entry`), both kinds `envkit.NilSeams` reports when nil.
The card names `TestWire_EverySeamFilled` in `internal/loomcli` and fills both fields in `loomCLI.wire`, but two other tests read every seam the same way:

- `internal/testkit/envkit/envkit_test.go` `TestFullEnv_FillsEverySeam` asserts `NilSeams(FullEnv(t))` is empty, and `FullEnv` (`internal/testkit/envkit/envkit.go`) fills neither field.
- `internal/battencli/wire_seams_test.go` `TestWire_EverySeamFilled` fails on any nil seam that is not in its `intentionallyNil` map, and batten's `wire` fills neither field.

Card gates run the whole untagged suite, as the plan's own `env-fields-and-wiring-land-together` decision says, so card 10 fails its gate.
Card 11 also depends on `FullEnv` filling `Seats` and a `Models` registry that resolves its model alias (see F2).

**Fix:** Add to card 10's `**Edit:**` `internal/testkit/envkit#FullEnv`, filling `Seats` with a `shedfake.SeatRunner` (card 8 lands earlier) and `Models` with a small registry literal, and `internal/battencli/wire_seams_test.go` (or `internal/battencli#intentionallyNil`), listing `Seats` and `Models` as loom-only seams batten never wires.
Raise the `internal/testkit/envkit` budget row only if a test is added, and none is needed.

### [BLOCKING] Card 11 misses shedbuild's every-registered-engine build test

**Location:** `_lyx/plan/11-shedrecipe-multillm-entry.md`

**Issue:** This is the focus directive's question, and the answer is no.
Card 11 lists `TestNames`, `minimalEntries` and `coverageGuardAllowedUnreachableEngines`, but `internal/shedbuild/build_engines_test.go` `TestBuild_EveryRegisteredEngineBuilds` ranges `shedrecipe.Names()` and builds each engine from `envkit.FullEnv` with the config from `engineMinimalConfig`.
That map has no `MultiLLM` key, so the row gets a nil config and `multiLLMEntry` rejects the missing `role`, `segment` and `seats` keys.
The test file's doc comment says this is deliberate, "so a newly registered engine fails this test until this fixture covers it."
Card 11's gate fails.

**Fix:** Add `internal/shedbuild#engineMinimalConfig` and `internal/shedbuild/build_engines_test.go` to card 11's `**Edit:**`.
Give the `MultiLLM` entry a chair and one advisor whose advisor output is a chair input, model aliases that `FullEnv`'s `Models` resolves (F1), and seat stencils the test writes into `env.StencilsDir` next to the existing `writeStencilFile` calls.
Each stencil includes a block that `FullEnv`'s stencils directory can read, or includes none.

### [MEDIUM] The planned include action passes no data to the block

**Location:** `_lyx/plan/02-stencil-include-form.md`; `_lyx/plan/03-seat-directive-stencils.md`

**Issue:** Card 2 has `FillWith` resolve a `{{template "name"}}` action, and card 3's block headers tell a seat stencil to pull blocks in with `{{template "seat-directive-chair"}}` and `{{template "seat-directive-advisor"}}`.
In `text/template` a `template` action with no pipeline executes the named template with nil data.
I checked this against the Go toolchain with a `missingkey=error` template matching `FillOptional`'s options: `{{template "blk"}}` fails with `executing "blk" at <.b>: nil data; no entry for key "b"`, and `{{template "blk" .}}` renders.
Both shipped blocks declare markers (`advisor_names`, `chair_name`, `output_files` and others), so every seat stencil written to the planned form fails at execution.
Card 2's marker check walks each associated template's tree without executing it, so it passes, and the failure surfaces only at `Execute`.
Card 2's own row, "an include with its own marker filled from the values", catches the failure when it is implemented, but the plan leaves the implementer to choose the remedy, and card 3's headers would document a form that cannot work.

**Fix:** In card 2's Intent, state one rule: either the include form is `{{template "name" .}}`, and `FillWith` refuses a `template` action whose pipeline is not `.` with an error naming the block, or `FillWith` passes the value map to a bare `{{template "name"}}` by setting the action's pipeline in the parsed tree before executing.
Add a refusal row or a rendering row for that rule to card 2's `TestFill` rows.
Make card 3's header wording, and the stencils cards 5, 12 and 13 write, follow the chosen form.

### [LOW] N = 1 is pinned against a fake engine, never the real one

**Location:** `_lyx/plan/09-shedadapters-multillm.md`; `_lyx/plan/06-seatengine-run.md`

**Issue:** D11 says "`seatengine` admits a table of one chair, and a test pins that such a table maps every shuttle outcome as `SingleLLMProducer` does."
Card 9 pins the equivalence with `MultiLLMProducer.Call` driven through `shedfake.SeatRunner`, whose `seatengine.Result` the test scripts.
That covers the adapter's mapping only.
Card 6's `engine_test.go` lists chair outcomes with advisors stopped, and no card runs `Engine.Run` or `Engine.Probe` over a table with no advisors (an empty advisor join, a notice sender with nothing queued, `failed_advisors` and `advisor_names` both `(none)`).

**Fix:** In card 6's `engine_test.go`, make the chair-outcome rows table-driven over a one-chair table as well as a chair-and-advisor table, so the engine's own result for N = 1 is pinned beside card 9's adapter comparison.
These are new rows in an existing test, and no new test function is needed.

## Focus verdicts

- Card 11 registry tests: not covered; see F2.
  `TestNames` and `minimalEntries` are listed, but shedbuild's every-engine build test is not.
- Card 6: a Done advisor queues nothing ("An advisor that reaches Done queues nothing").
  A seat whose `checkName` fails has its own strand stopped through `stopSeat` before the start error is recorded, with `ErrSeatNotStopped` on a failed stop.
  A chair `Wait` error has a stated outcome: `join` stops the chair and every live advisor and returns the wait error, or `ErrSeatNotStopped`.
  No defect.
- Cards 3 and 5: card 3's Intent states what the advisor-to-chair route lets through (any name `ListAgents` shows) and what bounds it (prompt naming, non-interactive advisors, quiet holds, advice-only weight).
  Card 5's Intent states that the chair-to-advisor edge lets an operator-grade instruction through, bounded by the advisor's write rule, its settings' denies and `Interactive` false.
  No defect.
- Card 5's refusal: the planned case is a seat stencil that declares a marker missing from its value map.
  `FillWith` refuses it through the unfilled-marker walk card 2 runs over the main template and every include (`unfilledTopLevelMarkers` treats an absent value as unfilled), and `missingkey=error` refuses a marker inside a branch.
  The refusal is built, so no defect.
  This holds independent of F3, which concerns execution, not the pre-execution check.
