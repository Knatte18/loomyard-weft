---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: contracts/specs/refusal-spec.md:158
    summary: The seat engine's new refusals (a held strand name, a seat that cannot be stopped) carry a way forward but have no row in the shed section and are not named in the spec's out-of-scope list
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shedadapters/doc.go:62
    summary: doc.go says an ErrSeatNotStopped error is returned without archiving, but one from Run follows the archive of every seat's stale outputs
---

Range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet` (default, `integration` and `llm` tags) over the touched packages, and `go test` over `seatengine`, `shedadapters`, `shedrecipe`, `stencil`, `shuttleengine`, `testkit/...`, `contracts/...` and `cmd/lyx` pass.
Top-level test counts match the `cmd/lyx/testdata/test-budget.yaml` rows for the packages the diff touches.

### [MEDIUM] Seat-engine refusals have no row in the refusal spec

**Location:** `contracts/specs/refusal-spec.md:158` (the `## shed` table), against `internal/seatengine/engine.go:207` and `internal/seatengine/engine.go:215`.
**Issue:** `checkName` (a strand name still held by an earlier strand) and `stopSeat` (`ErrSeatNotStopped`) are new refusals that end a shed step, and both carry a trailing `way forward:` clause.
The `## shed` table already holds the producers' strand-spawn refusals, and `PATTERN-refusal-way-forward` requires a new refusal in a bound module to land with its row in the same commit.
The diff adds no row, and the `## Out of scope` list names landing, batten, orch, fabric, burler and board but not seats, so the spec neither covers nor excludes them.
Whether `seatengine` is a bound module is the open choice: the refusals surface only through a `MultiLLM` row run by shed, and no loom row uses the producer yet, so the plan's cards leave the spec untouched.
**Fix:** Decide one way and record it in `contracts/specs/refusal-spec.md`: either add a row for each refusal (class correctness halt, quoting the `way forward:` clauses `lyx reed remove --name <name>` and `lyx reed remove <guid>`, then re-step the row), or name the seat engine in `## Out of scope` beside burler with the reason.

### [LOW] doc.go claims an ErrSeatNotStopped from Run is returned without archiving

**Location:** `internal/shedadapters/doc.go:62`
**Issue:** The sentence "an error wrapping seatengine.ErrSeatNotStopped keeps its way forward and is returned without archiving" holds for a probe or resume error.
`MultiLLMProducer.Call` archives every seat's stale outputs before it calls `Run`, so an `ErrSeatNotStopped` from `Run` is returned after the archive.
`TestMultiLLMProducer_ErrorExits` pins that: its "a run that cannot stop a seat" row has `wantArchived: true`, beside the probe row with `wantArchived: false`.
**Fix:** Reword the sentence to say the probe's `ErrSeatNotStopped` halts before the archive, and a run's keeps its way forward after it.

### Verdicts on the focus entries

1. Advisor loop in `Engine.Run` and `startSeat` (`engine.go:139-159`, `engine.go:191-197`): not a defect.
   An advisor whose strand took a held name and could not be stopped makes `startSeat` return `errors.Join(err, stopErr)`, which wraps `ErrSeatNotStopped` and carries `lyx reed remove <guid>`.
   `Run` then stops the notice sender and every already-started advisor through `stopAdvisors`, and returns that joined error; no live strand is left without a handle or a named way forward.
   The "an advisor's held strand name and a failed stop" row of `TestEngine_Run_StartFailureEndingTheRunStopsTheStartedAdvisors` reaches it, asserts both advisors stopped once and the chair never started.
2. `errorExit` over a cancelled context (`multillm.go:89-94`): not a defect, and a deliberate departure from the card's literal order.
   Card 9 says a cancelled context returns the context error and, separately, that an `ErrSeatNotStopped` error is returned without archiving; it does not say which wins when both apply.
   The code lets `ErrSeatNotStopped` win, so a seat that may still be live is not reported as a plain cancellation, and `errorExit`'s doc comment and `shedadapters/doc.go` both say so.
   The path is reached by the "a cancelled run that cannot stop a seat keeps its way forward" row of `TestMultiLLMProducer_Cancellation`; `TestMultiLLMProducer_ErrorExits` reaches the same error over a live context, so the cancelled-context row sits in the cancellation test rather than the one the directive names.
3. `doc.go` sentence on `ReservedMarkers` (`internal/seatengine/doc.go:21`): matches the code.
   `Table.Validate`'s collision check (`validateValues`) and `seatValues` read the unexported `reservedMarkers` declaration; the exported `ReservedMarkers` is read only by tests, and `entries_multillm.go` calls `Table.Validate` and reads no list itself.

No focus departures.
