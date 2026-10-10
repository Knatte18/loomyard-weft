# MultiLLMProducer: a chair and its advisors run as seats in one step

`shedadapters.MultiLLMProducer` is a fifth Shed producer, beside `SingleLLMProducer`.
It runs N agents in one step, each in a strand of its own.
One seat is the `chair`: its turn end, outputs and gates decide the step's outcome exactly as a single agent's do today, and a message to the producer is a message to the chair.
Every other seat is an advisor, `advisor-1` to `advisor-n`, whose outputs are the chair's inputs and who reach the chair over a channel the parent never sees.
Shed sees one step with one outcome.

The producer is built standalone.
No loom recipe row, Burler round or Discussion uses it yet.
Each of those is its own board entry.

## What changes

- New package `internal/seatengine` starts, joins, probes, resumes and stops the seats of one table over shuttle.
  It validates the table (one chair, advisors numbered from 1 in order, distinct outputs, every advisor output among the chair's inputs), composes each seat's spec and prompt, and sends the chair a notice line when an advisor ends.
- `shedadapters.MultiLLMProducer` maps the chair's result onto the Shed producer contract.
  A one-chair table maps every outcome as `SingleLLMProducer` does.
- `internal/shedrecipe` gains a `MultiLLM` registry entry whose row config carries the seat table, and `Env.Seats` and `Env.Models` seams that loom's wiring fills.
  A seat's `model` key uses model-spec notation.
- `internal/stencil` gains an include form (`FillWith`, `IncludeNames`), so a seat's one stencil pulls shared blocks in by name.
  The change touches a leaf every stencil passes through; `TopLevelMarkers` and every registered stencil's fill are pinned unchanged.
- Two shared blocks, `seat-directive-chair` and `seat-directive-advisor`, under `contracts/stencils/seat/`, carry the channel rules.
- `internal/shuttleengine` gains `Spec.QuietHold`, which turns the held-turn parent notice off for advisor seats, and one send lock per `Run`, so `Run.Send` calls and the gate's send on one run never type at once.
- Docs: `doc.go` for `seatengine`, `shedadapters` and `shedrecipe`, the `docs/overview.md` module table, `shed-recipe-spec.md` and `refusal-spec.md`.

## Design decisions

- Advisors start in table order and the chair last.
  Chair Done maps to Done with the chair's first output as the pointer; a failing chair gate maps to Stuck; chair death or timeout is an error.
  Live advisors are stopped on every terminal chair result.
- An advisor that fails to start, dies or times out does not fail the step.
  The chair is told (`failed_advisors` at start, one notice line once idle afterwards), and the result records it.
- The channel is Claude Code's `SendMessage`, so a second engine would need its own channel words in the seat blocks; `creel` removes that dependence later.
- A resume stops a finished advisor's kept strand, so a chair resumed mid-dialogue cannot ask it again; the chair is told, and the advisor's output stands.
- Cancellation returns the context error and stops nothing; a failed `Stop` returns an error naming the way forward and is not retried.
- Advisors' specs carry `KeepPane`; the chair's does not.

## Known risks, accepted

- An advisor hang is invisible to the engine; the chair's directive is the only bound.
- An advisor death notice lands only when the chair idles, so a chair that drafts straight through to its outputs never hears it.
- An advisor that disobeys its block can message any strand `ListAgents` shows; only review catches it.

## To check by hand

- Run the `llm`-tier smoke (`internal/seatengine/smoke_seats_test.go`): the advisor answers over the channel, the chair finishes, the advisor is stopped.
  It was not run.
  If delivery into a running chair turn proves unreliable, the fallback is the chair's directive telling it to end its turn after asking.
- `notice-stop-not-off-gate-boundary` in `internal/seatengine`: check that an advisor's notice, the gate's re-prompt and a pending send at the chair's output boundary never type at the same time, and that the gate spends no attempt on a notice.
- `seatengine-refusals-missing-from-refusal-spec` in `contracts/specs/refusal-spec.md`: check that the shed section lists the seat engine's held-name and not-stopped refusals.
- `internal/shedrecipe/seam_enforcement_test.go`, `registry_test.go`, `conformance_test.go` and `testdata/multillm-recipe.yaml` were edited beyond the seat work's own files; confirm the edits only register the new row.
- Run `go test ./...` and `go test -tags integration ./...`; the integration tier was not run for this description.
