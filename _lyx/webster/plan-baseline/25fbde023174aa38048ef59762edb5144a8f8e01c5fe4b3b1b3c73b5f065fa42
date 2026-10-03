# Card 6 — halt-notes

**Edit:**
- `internal/loomcli/arm.go`
- `internal/loomcli/run.go`
- `internal/loomcli/friction_test.go`
- `internal/loomcli/approve_awaiting_test.go`

**Create:**
- `internal/loomcli/halt.go`
- `internal/loomcli/halt_test.go`

**Uses:**
- `internal/loomcli#loomCLI.reflectFriction`
- `internal/friction#NotePath`
- `internal/logger#TraceFile`
- `internal/state#ReadJSONStrict`
- `internal/shedengine#Status`
- `internal/shedengine#StateFailed`
- `internal/shedengine#ErrShedBusy`
- `internal/shedverbs#Hooks`
- `internal/loomshed#NameWebster`
- `plan:internal/websterengine#WriteRefusalNote`
- `plan:internal/websterengine#RefusalNote`
- `internal/testkit/shedfake#Shuttle`

**Intent:**
A `step` or `run` invocation that leaves the run `blocked` or `failed` writes one Go-authored halt note and then runs the non-waiting reflection (D2), so halt friction is recorded mechanically and filed at the halt, under both verbs.

`halt.go`:

- `haltNote` holds the producer, the state, the reason, the history length and the trace file path.
- `writeHaltNote(frictionDir string, n haltNote) error` writes it through `friction.NotePath(frictionDir, "loom-halt")`, a no-op on an empty directory.
- `(c *loomCLI) reflectHalt(n haltNote) string` returns `StatusSkipped` on an empty friction directory; otherwise it writes the note (a write failure only logs) and returns `c.reflectFriction(false)`, which skips on a held reflection lock as the blocked path does today.
- `(c *loomCLI) failedHalt(err error) (haltNote, bool)` reports a `failed` halt behind a non-nil error: never for `shedengine.ErrShedBusy`, and otherwise only when the status file reads `failed` with a persisted error equal to `err.Error()`, which is how `shedengine` persists the producer-error and unrecognised-outcome arms.
  A stale `failed` status left by an earlier invocation therefore never writes a second note.
- `(c *loomCLI) loomAfterStep(ctx context.Context, res shedengine.StepResult, stepErr error) string` is loom's `AfterStep` hook:
  a `failed` halt behind `stepErr` reflects through `reflectHalt` and reports its status, which card 2 puts on the error envelope;
  `blocked` reflects through `reflectHalt` with `res.Producer`, `res.Reason` and `len(res.History)`;
  `done` reports `c.rowFrictionStatus`, or `skipped` when the row did not run in this process;
  every other state, `awaiting` and `paused` included, writes nothing and reports `skipped`.

`specFor` wires `AfterStep: c.loomAfterStep`.
`loomPostRun` gains the same halt handling: on `RunBlocked` it calls `reflectHalt` with the halted producer, reason and history; on a non-nil `runErr` it calls `reflectHalt` when `failedHalt` reports one, after `detectAndFileAnomalies` as today, and returns the status as the `friction` key, which card 2 merges onto `run`'s error envelope; any other `runErr` reports `skipped`.
`run` therefore now also reflects on `failed`, which it did not before.

What this lets through, and the bound: the reflection agent now files public issues at every `blocked` and `failed` halt under both verbs, where before only `run`'s `blocked` path reflected.
The halt envelope is delayed by at most `friction_timeout_min`, a held reflection lock skips the reflection rather than waiting, filing goes only through `lyx selfreport create` to the hardcoded `Knatte18/loomyard` repo, and the reflection stencil's content rule (card 8) bounds what an issue body may quote from the halt note.
`awaiting`, `paused`, a busy error and a stale `failed` status write nothing and spawn nothing.
`shouldReflectFriction` is removed, its decision now living in these two hooks, along with both tests that call it: `TestShouldReflectFriction` in `friction_test.go` and `TestShouldReflectFriction_AwaitingNeverReflects` in `approve_awaiting_test.go`, whose awaiting property moves to `halt_test.go`; the `run.go` and `loomPostRun` doc comments are updated to match.

`halt_test.go` covers, over a receiver with a temporary location, seeded stencils, Tier 2 on, the `selfreport` knob off (so `loomPostRun`'s `detectAndFileAnomalies` files nothing) and a `shedfake.Shuttle` answering `OutcomeDone`:

- a blocked step result writes a halt note naming the producer and reason, spawns one reflection, and reports `reflected`;
- a producer error whose status reads `failed` writes a halt note, reflects and returns `reflected`; a busy error and a stale `failed` status whose error differs write nothing and return `skipped`;
- a `done` result after the row recorded `failed` on `rowFrictionStatus` returns `failed`, and one where the row did not run returns `skipped`;
- `awaiting` and `paused` results write no note and spawn nothing;
- an unwritable friction directory still returns a status and never panics;
- `loomPostRun` writes a halt note, reflects and returns `friction: reflected` for `RunBlocked` and for a failed `runErr`, and for `RunAwaiting` writes no note, spawns nothing and returns `friction: skipped`;
- the done-when scenario: the generic `step` body (`shedverbs.Verbs`) over `c.specFor("step")`, with `PreStep` cleared and `BuildShed` replaced by a one-row shed whose `Webster` producer writes a `begin-batch` refusal note carrying `plan_drifted` through `websterengine.WriteRefusalNote` and returns Stuck, ends that step `blocked` with one reflection spawn whose prompt lists both the refusal note and the halt note, and with the envelope's `friction` reading `reflected`.
  The `shedfake.Shuttle` stands in for the agent and so for its `lyx selfreport create` filing, which makes it the scenario's fake issue filer; under `step` no Go filer runs.
- a second scenario over the same body whose producer returns an error ends that step with an error envelope whose `friction` reads `reflected`.

**ImpactSummary:** confined to `internal/loomcli`: `loomPostRun`'s halt branch, one new hook wired in `specFor`, and the removal of `shouldReflectFriction`, whose only production caller is `loomPostRun` and whose only other callers are one test each in `friction_test.go` and `approve_awaiting_test.go`.
