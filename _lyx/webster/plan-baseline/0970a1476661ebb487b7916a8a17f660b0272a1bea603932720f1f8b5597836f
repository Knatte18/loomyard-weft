# Card 4 — escalation-halt-skips-reflection

**Edit:**
- `internal/loomcli/halt.go`
- `internal/loomcli/halt_test.go`
- `pattern/PATTERN-friction-capture.md`
- `docs/overview.md`

**Intent:** D3: a halt classified `escalation-to-human` writes its halt note and spawns no reflection session.
The producer's own session holds the open question, and a second session reflecting on it looks like the run moving on and finds nothing but a process signal.

In `loomCLI.reflectHalt`, after `writeHaltNote`, classify the halt with `loomengine.ClassifyHalt(n.State, n.Reason)`; when the kind is `loomengine.AnomalyEscalation`, log at Info that the reflection is skipped for an escalation halt, naming the producer, and return `frictionengine.StatusSkipped` without calling `reflectFriction`.
`bounce-budget-exhausted` and `producer-hard-failure` halts reflect as today.
Both callers, `loomAfterStep` under `step` and `loomPostRun` under `run`, go through `reflectHalt`, so the envelope's `friction` key reports `skipped` for an escalation halt under either verb.
Update `reflectHalt`'s doc comment and the file comment of `halt.go` ("A halt has one path") to say an escalation halt writes its note and skips the reflection.

Bound, stated in the doc comment: only the `escalation-to-human` kind skips, and the skipped reflection is the only thing lost; the note stays on disk unarchived until a later reflection (the terminal Friction-Reflect row of the resumed run, or the next halt's reflection) covers it, so `PATTERN-friction-capture` holds.

Add one sentence to `pattern/PATTERN-friction-capture.md`'s halt bullet: an escalation halt writes its note but spawns no reflection, and the next reflection covers the note.
`docs/overview.md`'s `internal/frictionengine` entry, which says loom runs the reflection "after a `blocked` or `failed` halt", says an escalation halt writes its note and runs no reflection.

Tests in `halt_test.go`:

- `TestStep_BlockedHaltReflectsOverRefusalAndHaltNotes` drives `step` through the generic step body with a producer that returns Stuck on a row with no `OnStuck`, which is the escalation arm; it becomes the CLI-level D3 test: the envelope's `friction` is `skipped`, the reflection shuttle never ran, and both `loom-halt.md` (carrying `anomaly: escalation-to-human`) and the webster refusal note are still on disk.
  Rename it to describe that behaviour.
- `TestStep_ProducerErrorReflectsOnTheErrorEnvelope` keeps proving that a `failed` halt reflects under `step`, and gains the assertion that the reflection prompt names `loom-halt.md`.
- The `Blocked` subtest of `TestLoomPostRun_HaltNotes` switches to a reason starting with `shedengine.ReasonBounceBudgetExhausted`, so it keeps proving that a budget-exhausted halt reflects, and the test gains an escalation subtest (any other blocked reason) asserting `skipped`, no shuttle spec and the note written.
  The test-suite prune deleted the `loomAfterStep` blocked test, and both callers share `reflectHalt`, so this one test carries the per-kind split for both verbs.
- `TestLoomAfterStep_UnwritableFrictionDirectory_StillReturnsAStatus` blocks with reason `stuck`, which is now an escalation and would skip before reaching the reflection; it switches to a `shedengine.ReasonBounceBudgetExhausted` reason so it keeps exercising the reflection past an unwritable friction directory.

**ImpactSummary:** One branch in loomcli's shared halt reflection; every escalation halt under run or step stops spawning a reflection, other halt kinds are unchanged.
