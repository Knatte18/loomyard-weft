# Card 4 — loop-status-stop

**Rename:**
- `internal/shedengine#missingStatusClause` -> `plan:internal/shedengine#MissingStatusClause`

**Create:**
- `plan:internal/shedverbs#statusStopWayForward` -> `func statusStopWayForward(spec *Spec, readErr error) string`

**Edit:**
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedverbs#readStatusAt` -> `func readStatusAt(statusPath, statusLockPath string) (shedengine.Status, bool, error)`
- `internal/shedverbs#readStatusFile`
- `internal/shedverbs#ReportLoopArmError`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#childResult`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#Spec`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Intent:** D5: the `--until-stop` loop tells an absent status file from an unreadable one at an iteration's top, and its stop names the status file, the loop log and a way forward split by cause.

`shedengine`'s `missingStatusClause` is exported as `MissingStatusClause`, unchanged in body, so shedverbs reuses the recipe's told reseed clause with its generic fallback; its three in-package callers, `Goto`, `Relaunch` and `Shed.stepLocked`, call the new name.

`readStatusAt` re-signs to also return the read error from `state.ReadJSONNoMkdir`: `found` false with a nil error is an absent file.
`readStatusFile` keeps its `(Status, bool)` form and drops the error, and `ReportLoopArmError`'s two calls discard it.
`LoopSpec` gains the field `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`, the read at the top of each iteration; nil selects `readStatusAt`, like the struct's other nil-selects-production seams.
`childResult` gains `statusStop bool`, set when the iteration stopped on its status read, and `statusErr error`, that read's error.

`runLoop`'s top-of-iteration read goes through that seam:
- Found: the child runs as today.
- Absent: no child runs, and the result carries the cause `status-missing` with `statusStop` set.
- An error wrapping `state.ErrDecode`: no child runs, and the cause is `status-unreadable: <error>`, the `<class>: <detail>` form of `start-failed: <error>`. There is no re-read, since `internal/state` writes by atomic rename under the status lock, so the file stays undecodable to this binary.
- Any other error, a read-lock or raw read failure: when the loop's one re-step is unused, it marks it used, sets `obj.Restep` to `{Producer: lastKnown, Error: "status-unreadable: <error>"}` with an empty transient, logs it and re-reads through the same loop top. A re-read that succeeds steps on as usual. One that fails stops with `status-unreadable: <error>`. When the re-step is already used, by a transient child failure that no success has cleared since, it stops at once with `status-unreadable: <error>` and the read-error way forward, with no re-read, since D5's re-read runs under the existing single re-step and that re-step is spent.

At each status cause the loop writes one `logger.Warn` naming the status path, and the error when there is one.
In the `LoopStopInterrupted` branch, a result with `statusStop` set uses `spec.Loop.LogPath` as the trace file handed to `Hooks.AfterInterrupt` and to `interruptedRecord`, as the dead-loop stop already does.
Its error text is `interruptedError`'s message, then `; status file <StatusPath>, loop log <LogPath>; `, then `statusStopWayForward(spec, res.statusErr)`.
`runLoop` skips `shedengine.WriteFailedStop` for such a result, so the status file is left as found: still absent, or byte-for-byte the file that failed to read.
`runLoop`'s doc comment, which says an interrupted stop on a run still reading running writes it failed, adds this exception.

`statusStopWayForward` returns the trailing `way forward:` clause by cause:
- Absent: `shedengine.MissingStatusClause(spec.MissingStatusWayForward)`, the recipe's own reseed, so `lyx loom start` for loom, `lyx batten run <slug>` for batten and the generic `lyx shed seed` clause otherwise, as refusal-spec's `status file missing` row gives it.
- Decode error: `way forward: re-run lyx shed step <run-id> --until-stop with a lyx binary that decodes the status file, as after a version skew where an older loop binary cannot decode a file a newer one wrote; otherwise report the stop to the run's parent with the status file and the loop log`.
- Read error: `way forward: re-run lyx shed step <run-id> --until-stop`.

Every named step is one an agent may run or one lyx does, so `PATTERN-no-denied-recovery` holds.
The halt note loom's `loomAfterInterrupt` writes then carries the cause as its reason and the loop log as its trace file, and its history stays empty since the hook re-reads the file; loomcli itself is unchanged.
`Hooks`' `AfterInterrupt` doc names the loop log as the trace file at a status-read stop, and `Spec`'s `MissingStatusWayForward` doc says the loop's status-missing stop reads it too, not goto alone.

Bound: the stop adds no route and removes no guard. It departs from `PATTERN-failure-stop-visible` for an absent or unreadable status only, where nothing can be written without destroying the evidence, so the run is not made non-running.
`lyx loom arbiter` still refuses either stop under its `status or envelope unreadable` row.

Tests, as subtests of `TestLoop_InterruptedChildFillsTheErrorForm`:
- "status file vanishes between steps" also sets `fx.spec.MissingStatusWayForward` to `way forward: told reseed` and captures the hook's arguments.
  It asserts the exact error `shedverbs: step B interrupted (status-missing); status file <StatusPath>, loop log <LogPath>; way forward: told reseed`, `trace_file` and the hook's trace file equal to `fx.spec.Loop.LogPath`, the status file still absent, and one child started.
- "an undecodable status file stops without a re-read": the first child continues to B, then writes `{` over the status file.
  It asserts the cause in the error starts `status-unreadable: ` and contains `decode failed`, the error contains `with a lyx binary that decodes the status file` and `report the stop to the run's parent`, `trace_file` equals `fx.spec.Loop.LogPath`, the status file still reads `{`, one child started and no `loop.restep`.
- "a read error that fails its re-read stops": `fx.spec.Loop.ReadStatus` always returns `errors.New("read lock unavailable")`.
  It asserts no child started, the error contains `status-unreadable: read lock unavailable` and `way forward: re-run lyx shed step run-1 --until-stop`, `trace_file` equals `fx.spec.Loop.LogPath`, `loop.restep` names that error, and the status file still reads running at A.
  This row alone pins the skipped `WriteFailedStop`: the real file is readable here, so an unskipped write would turn it `failed`, while at an absent or undecodable file the write fails on its own.
- "a read error that clears on the re-read steps on": `fx.spec.Loop.ReadStatus` fails its first call and then delegates to `readStatusAt`, with one script halting the run done.
  It asserts the stop is halted, one child started and `loop.restep` names the read error.
- "a read error after a used re-step stops without a re-read": the first child returns a transient error envelope through `failing`, and `fx.spec.Loop.ReadStatus` delegates its first call to `readStatusAt` and then returns `errors.New("read lock unavailable")`.
  It asserts one child started, the error contains `status-unreadable: read lock unavailable` and `way forward: re-run lyx shed step run-1 --until-stop`, `ReadStatus` was called twice, and `loop.restep` names the child's transient error rather than the read error.

Docs:
- `contracts/specs/refusal-spec.md`'s shed section:
  - The `step interrupted` row, transient, drops "or the status file vanished" and adds `status-unreadable: <error>` from a read error that failed its one re-read or met a re-step already used, at which the status file is left as found rather than written `failed`. Its way forward names the status file, the loop log and re-running `lyx shed step <run-id> --until-stop`.
  - A new row, `loop status missing or undecodable`, a correctness halt like the `status file missing` row because neither cause clears by itself or by a retry. It names `status-missing` and `status-unreadable: <decode error>` at an iteration's top, no child run, the loop log as the trace file and the status file left as found, and gives each cause's way forward above.
- `internal/shedverbs/doc.go`'s stop paragraph states the two causes, the one re-read of a read error and the stop without it once the re-step is used, the loop log as the trace file, the split way forward and that the status file is left as found.
- `pattern/PATTERN-failure-stop-visible.md` gains a bullet stating the departure: an absent or unreadable status file at an iteration's top is left as found, since `shedengine.WriteFailedStop` cannot write over it and overwriting would destroy the evidence.

**ImpactSummary:** the loop's stop at a status read changes cause, way forward, trace file and status write; `readStatusAt`'s two other callers only drop the new error, and the clause rename is local to `shedengine`'s three callers.

**Commit:** 4: The shed loop tells an absent status file from an unreadable one and names its log (closes #581)
