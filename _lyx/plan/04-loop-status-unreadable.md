# Card 4 — loop-status-unreadable

**Create:**
- `plan:internal/shedverbs#statusStopWayForward` -> `func statusStopWayForward(spec *Spec, cause string, decode bool) string`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Uses:**
- `internal/shedverbs#interruptedError`
- `internal/shedverbs#interruptedRecord`
- `internal/shedverbs#newLoopFixture`
- `internal/shedverbs#Spec`

**Intent:** D5: the loop tells an absent status file from an unreadable one and points at its own log.

`LoopSpec` gains one field, `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`, the read the loop makes at the top of each iteration; nil selects `state.ReadJSONNoMkdir[shedengine.Status]`.
It exists so a test can fail one read and let the next pass; production never sets it.

`runLoop`'s top-of-iteration read stops using `readStatusFile`'s collapsed bool and calls the `ReadStatus` seam, then:
- found: steps the child as today.
- not found with a nil error (absent file): the cause is `status-missing`.
- an error wrapping `state.ErrDecode`: the cause is `status-unreadable: <error>`; no re-read, since `internal/state` writes by atomic rename under the status lock and a file that fails to decode stays undecodable to this binary.
- any other error (the read lock or a raw read, `state.ErrRead`): transient; when the loop's single re-step is unspent (`restepped` false) it sets `restepped`, records nothing in `obj.Restep`, logs and re-reads at once without starting a child; when the re-read fails too, the cause is `status-unreadable: <error>`.
  A re-read that finds the file lets the loop step on as normal.
At each of these causes the loop writes one `logger.Warn` naming the status path and, for `status-unreadable`, the error, and runs no child.

At an interrupted stop whose cause is `status-missing` or `status-unreadable: …`, `runLoop`:
- builds the message with `interruptedError` as today and the way forward with `statusStopWayForward` instead of `interruptedError`'s generic re-run, keeping the `<message>; way forward: <clause>` form;
- hands `spec.Loop.LogPath` as the trace file to `Hooks.AfterInterrupt` and to `interruptedRecord`, as the `loop-exited` stop already does, instead of the first child trace file;
- skips `shedengine.WriteFailedStop`, so the status file is left exactly as found and the stop is not made non-running: overwriting an unreadable file would destroy the evidence and an absent one cannot be written; this is the stated departure from `PATTERN-failure-stop-visible` for the case where no status can be read.
Every other stop keeps its current behaviour.

`statusStopWayForward` (in `internal/shedverbs/loopenvelope.go`, beside `interruptedError`) returns the clause without the `way forward: ` prefix, always naming the status path and the loop log (`spec.Loop.LogPath`):
- `status-missing`: the recipe's own reseed, `spec.MissingStatusWayForward` with its leading `way forward: ` trimmed (`lyx loom start` for loom, `lyx batten run <slug>` for batten), and with it empty `run "lyx shed seed <run-id> --recipe <name>"` first.
- `status-unreadable` from a read error that failed its re-read (`decode` false): re-run `lyx shed step <run-id> --until-stop`.
- `status-unreadable` from a decode error (`decode` true): re-step with a binary that decodes the file, as after a version skew where an older loop binary cannot decode a file a current binary reads, with `lyx shed step <run-id> --until-stop`; otherwise report the stop to the run's parent with the status path and the loop log.

The halt note loom writes through `loomAfterInterrupt` then carries the cause as its reason and the loop log as its trace file; its history stays empty, since the hook re-reads the file.
A file that later reads `running` is resumed by the way forward's re-step, not by `lyx loom resume`.

Tests, as subtests of `TestLoop_InterruptedChildFillsTheErrorForm` over `newLoopFixture`, each with an `AfterInterrupt` hook recording its arguments:
- "status file vanishes between steps" (existing) also sets `spec.MissingStatusWayForward` to `way forward: run "lyx loom start" first to bootstrap this task`, and asserts the error holds `step B interrupted (status-missing)`, `lyx loom start`, the status path and the loop log path, the hook's file and the envelope's `trace_file` are `spec.Loop.LogPath`, one child ran, and the status file is still absent.
- "status file absent with no told reseed": the first child removes the status file and `MissingStatusWayForward` is empty; the error names `lyx shed seed`.
- "status file holds invalid JSON": the first child continues to B and then overwrites the status file with `{not json`; the stop is interrupted with a cause starting `status-unreadable: `, the error names `lyx shed step run-1 --until-stop`, a binary that decodes the file, and reporting to the run's parent, one child ran, the file's bytes are unchanged, and the hook's file is the loop log.
- "a read error that clears on the re-read steps on": `Loop.ReadStatus` fails its first call at the top of the second iteration with an error wrapping `state.ErrRead` and delegates otherwise; scripts `continuing("B", 1)` then `halting(shedengine.StateDone, "", 2)`; the loop halts done after two children.
- "a read error that fails the re-read stops": `Loop.ReadStatus` fails every call after the first child with an error wrapping `state.ErrRead`; the stop is interrupted with a cause starting `status-unreadable: `, the way forward names `lyx shed step run-1 --until-stop`, one child ran, and the status file still reads running.

Docs in the same commit:
- `internal/shedverbs/doc.go`'s loop section replaces "including a status file that vanished" with the two causes, the single re-read of a raw read error, the loop log as trace file, the split way forward, and that these stops leave the status file as found.
  Its sentence "A transient error gets one immediate re-step" states that a raw status read error spends that same single re-step on its re-read, so a transient child failure right after a cleared read error stops the loop instead of re-stepping.
- `pattern/PATTERN-failure-stop-visible.md` gains a bullet beside the busy and Go-driver exclusions: a loop stop at `status-missing` or `status-unreadable` stays outside the invariant, since no status can be read to write `failed` over and overwriting an unreadable file would destroy the evidence; such a run may still read `running`, and the stop's way forward names a re-step rather than `lyx loom resume`.
  The bullet "`lyx loom resume` therefore never answers \"already running\" for such a stopped run" keeps its scope to the stops the invariant covers.
- `contracts/specs/refusal-spec.md`'s `## shed` table: the `step interrupted` row's trigger drops "or the status file vanished" and adds `status-unreadable: <error>` from a read-lock or raw read error that failed its one re-read, at which the status file is left as found rather than written `failed`, with the way forward re-run `lyx shed step <run-id> --until-stop`, class still transient; a new row after it, `loop status missing or undecodable` | the loop finds no status file (`status-missing`) or one that does not decode (`status-unreadable: <error>`) at the top of an iteration; no child runs, the trace file is the loop log and the status file is left as found rather than written `failed` | correctness halt | at `status-missing` the recipe's own reseed, `lyx loom start` for loom, `lyx batten run <slug>` for batten, `lyx shed seed` otherwise; at a decode error re-step with a binary that decodes the file, otherwise report the stop to the run's parent with the status path and the loop log.

**ImpactSummary:** `runLoop`'s top-of-iteration read and its interrupted stop at the two status causes change; child-step stops, `loopExitStop` and every other `readStatusFile` caller keep their behaviour, and `LoopSpec` gains one optional seam no production arming sets.

**Commit:** `4: shed loop tells an absent status file from an unreadable one (#581)`
