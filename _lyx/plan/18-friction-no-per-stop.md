# Card 18 — friction-no-per-stop

**Edit:**
- `internal/loomcli#loomCLI.reflectHalt`
- `internal/loomcli#loomCLI.loomAfterStep`
- `internal/loomcli#loomCLI.loomPostRun`
- `internal/loomcli#loomCLI.loomAfterInterrupt`
- `internal/loomcli#loomCLI.reflectFriction` -> `func (c *loomCLI) reflectFriction() string`
- `internal/loomcli#loomCLI.reflectFrictionRow`
- `internal/loomcli#awaitRunLock`
- `internal/loomcli#dispositionForHandshake`
- `internal/loomcli#TestLoomPostRun_HaltNotes`
- `internal/loomcli#TestStep_EscalationHaltSkipsReflectionAndKeepsBothNotes`
- `internal/loomcli#TestReflectFriction_DepsValidationFailureReportsFailed`
- `internal/loomcli/doc.go`
- `internal/loomengine#LoomFrictionLock`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `PATTERN.md`
- `pattern/PATTERN-friction-capture.md`

**Delete:**
- `internal/loomcli#TestReflectFriction_SkipsWhenAnotherDriverHoldsTheReflectionLock`
- `internal/loomcli#TestStep_ProducerErrorReflectsOnTheErrorEnvelope`

**Create:**
- `plan:internal/loomcli#TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` -> `func TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection(t *testing.T)`
- `plan:internal/loomcli#TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection` -> `func TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection(t *testing.T)`

**Uses:**
- `internal/loomcli#haltFixture`

**Intent:** D8: no friction session per stop.

`reflectHalt` writes the `loom-halt` note through `friction.NotePath` as today, with the parent's dirty-tree section, and reports `skipped`; it spawns no reflection, so the escalation special case goes.
Its callers, `loomAfterStep` under step, `loomPostRun` under run and `loomAfterInterrupt` under the loop, keep writing the note and report `skipped` for a `blocked` or `failed` halt; their doc comments and `halt.go`'s file header say the note waits for the run's Friction-Reflect row, which reflects every note in the friction directory as today.
So no stop holds the driver lock for a friction session and `lyx loom resume` never waits on one.
`reflectFriction` loses its `wait` parameter: the halt path that skipped on a held lock is gone, and the row path always waits; `reflectFrictionRow` calls it, and `run.go`'s header and comments drop the halt path.
`friction_timeout_min` stays, bounding the row's reflection and lock wait.
`awaitRunLock`'s doc, which names a reflection after a blocked halt, and the `awaitRunLockHalted` paragraph of `dispositionForHandshake`'s doc, which says a blocked halt runs the friction reflection after `shedengine.Run` released the lock, are rewritten: the arm stays, since it also fires on the first poll of every resume of a halted run.
`LoomFrictionLock`'s doc, which justifies a lock of its own by the blocked-halt reflection running outside the run lock, and the comment in `runDriverSpawnAndWait` naming a blocked halt's reflection, are rewritten: the lock now orders the Friction-Reflect row's reflections of two drivers over one friction directory.
`TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection` (no test reaches `loomAfterInterrupt` today): the hook writes the `loom-halt` note with the cause as reason, reports `skipped` and runs the reflection shuttle zero times.

`PATTERN.md`'s entry becomes the declared text:

```markdown
- `PATTERN-friction-capture` — Halting a loom run or refusing in webster: Go writes a friction note, reflected by the run's Friction-Reflect row rather than at the stop, and archived only once a reflection covers it. (test) — [background](pattern/PATTERN-friction-capture.md)
```

and its background file says a halt note is reflected at the run's end by Friction-Reflect, not at the stop, keeping the rule that no note is archived before a reflection covers it.

Tests: `TestLoomPostRun_HaltNotes` asserts each halt writes its note, reports `skipped` and runs the reflection shuttle zero times; `TestStep_EscalationHaltSkipsReflectionAndKeepsBothNotes` keeps both notes with no spawn; `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` replaces the producer-error test: the error envelope's friction key is `skipped`, the `loom-halt` note is on disk and no reflection ran; `TestReflectFriction_DepsValidationFailureReportsFailed` calls the parameterless form; the held-lock skip test goes with the path it pinned, while `TestReflectFrictionRow_WaitsOnAHeldReflectionLock` keeps the row's wait covered.

**ImpactSummary:** Loom's four halt hooks stop spawning reflections and reflectFriction loses its unused parameter; callers are all in loomcli.

**Commit:** 18: loom: a halt writes its friction note and leaves the reflection to Friction-Reflect
