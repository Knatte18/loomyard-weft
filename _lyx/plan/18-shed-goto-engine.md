# Card 18 — shed-goto-engine

**Create:**
- `internal/shedengine/goto.go`
- `internal/shedengine/goto_test.go`

**Uses:**
- `internal/shedengine/producer.go`
- `internal/shedengine/run.go`

**Intent:**
Add the status-file mutation behind `lyx shed goto`, in the package that owns the status file's writes, so the generic verb in `internal/shedverbs` (card 20) stays a thin body that derives no path.

`goto.go` declares `GotoRequest` (`StatusPath`, `LockPath`, `StatusLockPath`, `Producers []ProducerDef`, `Target string`) and `Goto(req GotoRequest) (Status, error)`.
All three paths are told; `Producers` needs only `Name` and `Segment`, so a `Routing.Producers` projection works.

`Goto`:

1. Creates both lock paths' parent directories, as `preflight` does, then takes the run lock non-blocking.
   A held lock refuses with an error wrapping `ErrShedBusy`: `way forward: "lyx shed pause" asks the live driver to stop at its next producer boundary; check the holder with "lyx shed status", then retry`.
2. Reads the status file strictly.
   A missing file refuses as today's step does; an invalid state refuses as today's step does.
3. `state: done` refuses: `goto never re-opens a finished run; way forward: seed a new run`.
4. A `Target` not in `Producers` refuses, listing every valid name in list order.
5. Otherwise writes, in one `state.UpdateJSON` with the vanished-file guard `persist` uses: `current_producer` = target, `state` = `paused`, `error` and `transient` empty, `pause_requested` false, one appended history entry `{producer: target, outcome: goto, at: now}`, and `activity` recomposed through `composeActivity`.
   It calls no `CommitStatus`: the next step's persist commits the file, as `pause`'s write is.
   It sets no transient mark.

`goto_test.go` (untagged, status files under `t.TempDir()`) covers: a blocked run moved to another row → the written file has the target, `paused`, the history entry and empty error; the run lock held by the test → `errors.Is(err, ErrShedBusy)`, the message names `lyx shed pause` and `lyx shed status`, the file unchanged; an unknown target → the message lists every valid name, the file unchanged; a `done` run → refused naming a new seed, the file unchanged; and a `Step` after a `goto` resumes at the target through the resume write.
