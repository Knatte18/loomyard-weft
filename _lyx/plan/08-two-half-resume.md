# Card 8 — Probe and resume a two-half round

**Create:**
- `internal/burlerengine/resume.go`

**Edit:**
- `plan:internal/burlerengine#RunnerShuttle`
- `internal/burlerengine/shuttle.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/doc.go`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/attachprobe_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/testkit/shedfake/shedfake.go`
- `internal/testkit/shedfake/shedfake_test.go`
- `internal/burlerengine/shuttle_test.go`
- `internal/testkit/envkit/envkit.go`
- `internal/shedrecipe/fixture_test.go`
- `internal/battencli/wire_seams_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#Runner.ProbeGated`
- `plan:internal/burlerengine#Handle`
- `plan:internal/burlerengine#StrandRemover`
- `plan:internal/burlerengine#ErrHalfNotStopped`
- `plan:internal/burlerengine#Engine.join`
- `internal/shedadapters#BurlerDeps`
- `internal/shedadapters#BurlerProducer.probeLiveRound`
- `internal/shedadapters#archiveStaleOutputs`

**Intent:**
Implement D6's resume and adapt `BurlerProducer`'s live-round probe and retry to two strands, replacing the pair-file attach probe card 7 left in place.

`burlerengine`.
`Shuttle` gains `ProbeGated(shuttleengine.Spec, shuttleengine.GateSpec) (Handle, bool, error)`;
`RunnerShuttle` implements it over `Runner.ProbeGated` and returns a nil `Handle`, never a typed-nil one, on the not-found answer.
`resume.go` adds a per-half state (gone, done, live), a `LiveRound` carrying each half's state and, for a live half, its `Handle` (from which its strand guid is read), and two `Engine` methods.
`ProbeRound(p Profile, opts RunOpts) (LiveRound, error)` validates `p`, builds both halves' specs and gate specs with card 7's helper, and probes each half by its own single output file without waiting:
live is a found run (a run whose record still reads running is live even when its output file is on disk, as a reviewer repairing its review in the parse gate is), done is no live run with the output file present, gone is neither.
A probe error is returned as is.
`Resume(p Profile, opts RunOpts, live LiveRound) (Result, error)` requires both halves live, removes any marker it finds (a marker here is stale, since a finished reviewer's run is never live), and hands both handles to `join`, so the two waits run concurrently, `reviewReady` runs when the attached reviewer completes, and the D7 comparison runs against the bytes it returned.
An attached fixer that completes before the marker, an attached half that dies or times out while the other is live, and a review changed after a resumed handoff end the round exactly as a fresh round ends.

`shedadapters`.
`BurlerRunner` gains `ProbeRound` and `Resume`.
`BurlerDeps` drops `Attach` and gains `Remover burlerengine.StrandRemover`, the same remover the engine is told (D6), required non-nil.
`Call` keeps "probe before archive" and replaces `probeLiveRound`:
with both halves live it calls `Resume` and maps the result through the same outcome switch as a spawned attempt, except that a died or timeout result falls through to a fresh attempt 1, as an attached died round does today;
with exactly one half live it removes that half's strand through `Remover`, and with one half done and the other gone (no half live) it removes nothing, and both fall through to attempt 1, whose archive renames the round's outputs and whose `Engine.Run` removes the marker;
with neither half live it falls through to attempt 1.
A probe error, a failed removal and a `Resume` error wrapping `ErrHalfNotStopped` return without archiving and without spawning.
The failed-removal return's message ends with the same clause as card 7's `ErrHalfNotStopped` error, `way forward: run "lyx reed remove <guid>", then re-step the row`, naming the live half's guid, and the loom refusal row card 7 widens covers it (`PATTERN-refusal-way-forward`).
What this lets through, and its bound: a live fixer beside a finished review is stopped and re-run rather than attached, so its target edits stay in the worktree and the next attempt's reviewer reviews them;
a round never ends with one half still live, and a half already live is never spawned a second time.

`shedrecipe`: `Env.BurlerRemover` carries the remover;
`burlerRoundEntry` requires it and no longer requires `Env.Shuttle`, which only the probe needed.
`loomcli/wiring.go` sets `Env.BurlerRemover` to the remover it passes to `burlerengine.New`.
The two shared Env fixtures fill the new seam, since `BurlerRound` now refuses an Env without it and `envkit.NilSeams` reports every nil interface field: `envkit.FullEnv` and `shedrecipe`'s `newTestEnv` (in `fixture_test.go`) set `BurlerRemover` to one `shedfake.StrandRemover` stub that records the guids it was asked to remove (the shared kit holds it, since `shedrecipe` cannot import `envkit`), and batten's `intentionallyNil` map in `internal/battencli/wire_seams_test.go` gains a `BurlerRemover` row with the same loom-only reason as its `Burler` row.
`shedfake.BurlerRunner` records `ProbeRound` and `Resume` calls and returns scripted `LiveRound`s and results.

Docs in this commit: `burlerengine/doc.go` gains the resume section;
`shedadapters/doc.go` describes the two-strand probe, the stop-and-rerun cases and the retry.

Tests.
`engine_test.go`, with card 7's fake handles, gains two new table-driven functions:
`ProbeRound` classifies live, done and gone per half, including a found run whose output file exists;
`Resume` attaches both with neither wait blocking the other, removes a stale marker, writes the marker only after the attached reviewer is done with a parsing review, spawns nothing, and has one failure row (an attached fixer that completes before the marker) proving the attached handles route into `join`;
the half-died, skipped-handoff and changed-review outcomes are pinned once, by card 7's failure-rules and marker-lifecycle tables, which `Run` and `Resume` share through `join`.
`shuttle_test.go` (card 7's function over the real adapter) gains a `ProbeGated` row over `RunnerShuttle(shuttleengine.NewRunner(…))` on `shuttlefake` seams with no candidate run: it returns an interface-nil `Handle` and `found` false, never a typed-nil handle.
`attachprobe_test.go` (the existing live-probe file) has its four `BurlerProducer` tests rewritten for two halves, covering both live resumes with no spawn, neither live spawning attempt 1, a resumed died outcome respawning from attempt 1, and a probe error or failed removal returning an error with nothing archived or spawned, the removal row asserting the way-forward clause;
one new table-driven function there covers a pair with one live half, which removes that strand and spawns attempt 1 with round token `"1"`, and a done/gone pair, which removes nothing and spawns attempt 1 the same way.
`burler_test.go` keeps only the rows its own round logic needs, with `TestBurlerProducer_Gate_ProbeLiveRoundPassesGateAndMapsFailedGateIdentically` rewritten for the resumed fixer's gate.
`entries_burler_test.go` asserts a missing `Env.BurlerRemover` is refused and a missing `Env.Shuttle` is not.
`wiring_test.go` asserts the Env remover is wired, `smoke_attachprobe_test.go` moves its runner to the new interface, and `shedfake_test.go` covers the new fake methods as rows of its existing `BurlerRunner` test.
The two new `engine_test.go` functions raise the `internal/burlerengine` row of `cmd/lyx/testdata/test-budget.yaml` by two;
the one new `attachprobe_test.go` function stays inside the `internal/shedadapters` row's headroom, so that row is unchanged (`TestTestBudget_PackagesStayWithinBudget` fails on a package over its row).

**ImpactSummary:** `BurlerRunner`, `BurlerDeps` and `shedrecipe.Env` change; the producer's resume path is replaced, its spawn path and retry stay, and the two shared Env fixtures, batten's seam test and the budget row follow the new `Env` field.
