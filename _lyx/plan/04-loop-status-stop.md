# Card 4 — loop-status-stop

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (shedengine.Status, bool, error)`
- `plan:internal/shedverbs#statusStopWayForward` -> `func statusStopWayForward(spec *Spec, cause string, decodeErr bool) string`

**Edit:**
- `internal/shedverbs#runLoop`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#loopRestep`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Uses:**
- `internal/state#ErrDecode`
- `internal/testkit/logcapture#Capture`

**Intent:** D5, #581: a loop that stops because it cannot read `status.json` at the top of an iteration says whether the file was absent or unreadable, and points at its own log.

`LoopSpec` gains a field `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`; nil selects `state.ReadJSONNoMkdir[shedengine.Status]`.
It is a test seam: no arming module sets it, so production always reads through `internal/state`, and it lets nothing through that the real read would refuse; it exists so a row can make a read error clear on the re-read.
`readLoopStatus` reads `spec.StatusPath` through it and returns the status, whether the file was found, and the error.
`runLoop`'s per-iteration read uses it in place of `readStatusFile`; the first read before the loop and the read after the stop keep `readStatusFile`.
- An absent file stops the loop with the cause `status-missing`, as today.
- An error wrapping `state.ErrDecode` stops it at once with `status-unreadable: <error>`, with no re-step: the only writer, `internal/state`, writes by atomic rename under the status lock, so the file stays undecodable to the binary that failed.
- Any other error, a read-lock or raw read failure, is transient: under the loop's existing single re-step (`restepped` false) the loop sets `restepped`, records `loopRestep{Producer: lastKnown, Error: <error>}` with an empty `Transient`, since no step classed it, logs the re-read and reads again without starting a child; a re-read that succeeds steps on, and a failure with the re-step spent stops with `status-unreadable: <error>`.
At either cause the loop writes a `logger.Warn` naming the status path and, when unreadable, the error; no child runs for that iteration.

At these two causes the stop is `interrupted` as today, with three differences.
The trace file handed to `Hooks.AfterInterrupt` and to `interruptedRecord` is `spec.Loop.LogPath`, as `loop-exited` already does, instead of a child trace that does not exist.
The error text keeps `interruptedError`'s message and replaces its way forward with `statusStopWayForward`, which names the status path and the loop log, then by cause:
- `status-missing`: the recipe's own reseed, `spec.MissingStatusWayForward` with its leading `way forward: ` removed (`lyx loom start` for loom, `lyx batten run <slug>` for batten), or, when it is empty, seeding the run with `lyx shed seed`, as refusal-spec's `status file missing` row gives it.
- `status-unreadable` from a read error that failed its re-read: re-run `lyx shed step <run-id> --until-stop`.
- `status-unreadable` from a decode error: re-step with a binary that decodes the file, as after a version skew where an older loop binary cannot decode a file a current binary reads, with `lyx shed step <run-id> --until-stop`; otherwise report the stop to the run's parent with the status path and the loop log, as refusal-spec's `arbiter: status or envelope unreadable` row does.
`shedengine.WriteFailedStop` is not called at these causes, so the status file is left as found: it cannot write a failed stop over an absent or undecodable file, and over a file that reads again by then it would overwrite the evidence.
The cause keeps the `<class>: <detail>` form, which `convergence-plan` reads by its leading clause; the halt note loom's `loomAfterInterrupt` writes carries the cause as its reason and the loop log as its trace file, and its history stays empty since the hook re-reads the file.

Bound: at these causes the status is not made non-running, a stated departure from `PATTERN-failure-stop-visible` for a status that cannot be read; a file that later reads `running` is resumed by the way forward's re-step, not `lyx loom resume`, and `lyx loom arbiter` refuses either stop under its `status or envelope unreadable` row.
`pattern/PATTERN-failure-stop-visible.md` gains a bullet stating that departure and naming the test below.

Tests: `TestLoop_InterruptedChildFillsTheErrorForm`'s `status file vanishes between steps` subtest becomes a table over the state child 1 leaves after its `continuing("B", 1)` step, each row with `AfterInterrupt` capturing its arguments and `logcapture.Capture` capturing the log, and asserting the stop's cause, `env["trace_file"]` and the hook's trace argument equal to `fx.spec.Loop.LogPath`, the error naming the status path and the loop log, a WARN line naming the status path, and the status file left as the row made it:
- the file removed, with `MissingStatusWayForward` set to loom's clause: cause `status-missing`, exactly one child started, the way forward naming `lyx loom start`, the file still absent;
- the file removed, with `MissingStatusWayForward` left empty as `newLoopFixture` leaves it: cause `status-missing`, the way forward naming `lyx shed seed`;
- the file overwritten with `{"current_producer":`: cause `status-unreadable:` naming the decode error, one child, no `loop.restep` key, which pins that a decode error is not re-read, the way forward naming a binary that decodes the file and reporting to the run's parent, the bytes unchanged;
- the file replaced by a directory, which fails both reads with a raw read error: cause `status-unreadable:`, one child, `loop.restep` naming the read error, the way forward naming `lyx shed step run-1 --until-stop`, the directory still in place;
- `ReadStatus` set to fail its first call after child 1 with an error and then read the real file, and a second scripted child `halting(shedengine.StateDone, "", 2)`: the loop stops halted with two children, and `loop.restep` names the cleared read error.
`internal/shedverbs/doc.go`'s stop vocabulary paragraph replaces "including a status file that vanished" with the two causes, the trace file and that the status file is left as found, and its sentence "A transient error gets one immediate re-step, reported as restep." gains that a status read error gets the same one re-read.
`loopRestep`'s doc comment says it also records the re-read after a status read error, with an empty `Transient`.
`contracts/specs/refusal-spec.md`'s shed section changes in two rows, each stating that at its causes the status file is left as found rather than written `failed` and giving its way forward above: the transient `step interrupted` row drops "or the status file vanished" and names `status-unreadable` from a read error that failed its re-read; a new `loop status missing or undecodable` row after it, a correctness halt like the `status file missing` row, names `status-missing` and `status-unreadable` from a decode error.

**ImpactSummary:** Changes only the loop's top-of-iteration status read and the interrupted stop it produces; `LoopSpec` gains one nil-defaulted seam field, so every arming module compiles unchanged.

**Commit:** `4: The loop tells an absent status file from an unreadable one and names its log`
