```yaml
lens: code-affected-set
outcome: reported
```

Task restated: lens code-affected-set, listing what reads or asserts each changed output outside the touched lines.

**Per change, the set found:**

- **`spawnLoop(spec, loopID, argv)` signature.**
  - Callers: `loopwait.go:112` and the new integration test at `loop_unix_integration_test.go:100`. Both are updated.
  - The `Spawn` seam signature is unchanged. Covered.
- **Dead-loop `loop.detail` and cause text** (`; loop exit: …`, `; loop log: …`, `; step stderr: …`).
  - Readers of the `loop-exited` string outside `shedverbs`: `internal/loomengine/anomaly_test.go:164`, `internal/loomcli/halt.go`, `internal/loomengine/anomaly.go`, `internal/sheddriver/drive_test.go:93`, `contracts/stencils/shed/shed-template-driver.md`, `pattern/PATTERN-shed-driver.md:54`.
  - All of them only match the `loop-exited` prefix or the interrupted kind. None asserts the old detail suffix. Covered.
  - In-package assertions are updated: `loopwait_test.go` rows now carry `loop exit: not recorded`, and the early-exit row at `loopwait_test.go:772` is updated. Covered.
- **`loop.stderr_path` and `loop.trace_copy` now filled on a dead-loop stop.**
  - Readers: `contracts/stencils/shed/shed-template-driver-guide.md:11,15`, `shed-template-driver.md:86`, `shed-template-tackler.md:7,46`.
  - These already tell the reader to open both paths, so the change makes them true and nothing is left wrong. Covered.
  - Writers of `StopFiles` are `internal/loomcli/looparm.go:50` and `internal/battencli/arm.go:531`, which the new code reads unchanged. Covered.
- **`LoopSpec.Signals` and `LoopSpec.Raise`, new optional seams.**
  - No other package sets them; a grep found no uses outside `shedverbs`.
  - A nil seam selects `os/signal`, so the arming modules (`looparm.go`, `arm.go`) and the other `runLoop` fixtures need no change.
  - `signal.Notify` and `signal.Reset` appear nowhere else in production outside `shedverbs`, so there is no second signal owner to conflict with. Covered.
- **`stderrExcerpt` split into `outputExcerpt`.** Both callers (the `stderrExcerpt` call in `deadLoopStop`, and `loopExitEvidence`) are in the diff. Covered.
- **New loop-log marks** (`lym shed: loop <id> …`).
  - The loop log's other readers: grepping `LogPath` and "loop log" outside `shedverbs` found only unrelated `SupportLogPath` and dobby log hits.
  - `statusStop` errors and `interruptedTraceFile` use `loop.LogPath` as a path, not parsed content. Covered.
- **Docs** (`internal/shedverbs/doc.go`).
  - The new paragraph and the updated dead-loop paragraph match the code: signal marks, panic recovery, `loop exit: not recorded`, and `stderr_path`/`trace_copy`.
  - `PATTERN.md` and `pattern/` are untouched, and the rubric's declared-changes list names none. Nothing to land.
- **Repository rules.**
  - `PATTERN-run-process-owner` says "a site that signals a run's process by any other route is a review finding" (`pattern/PATTERN-run-process-owner.md`). `raiseSignal` (`loop.go:463`) signals the loop's own process to re-raise a signal it received, which is not a route for ending another run process. I do not count it as a violation.
  - `PATTERN-spawn-observability` is met: the spawn is still logged through `logger`.
  - `PATTERN-named-constants` is met: `loopLogMarkPrefix` and `loopStartingMark` are declared once and used by the writer and the reader.
  - `PATTERN-shed-driver` allows generic verb bodies in `internal/shedverbs`, which is where this change sits.
- **Commit message.** It carries no `Pattern-Departure:` trailer and needs none, since no entry is departed from. Its claims match the diff.

**Findings:** none from this lens.

**Tool calls (6):**
1. `grep spawnLoop(` plus a grep for `loop-exited` and detail text across the repo: to find callers and readers outside `shedverbs`.
2. `grep stderr_path|trace_copy` in docs and stencils, plus `Signals|Raise` uses: to find readers of the filled paths and outside users of the new seams.
3. `grep` for `LogPath` outside `shedverbs`, plus `pattern/PATTERN-run-process-owner.md`: to find other readers of the loop log and the rule text.
4. `grep` of the `loop-exited` and `ended without an envelope` hits in `anomaly.go`, `halt.go`, `spec.go`, the driver stencil, `PATTERN-shed-driver.md` and the tests: to see whether anything parses the old detail text.
5. `grep` for `signal.Notify`, `signal.Reset` and `FindProcess` across production code, plus `spec.go` for `Signals|Raise|LogPath`: to check for a second signal owner and whether `spec.go` needs the new seams. The `spec.go` grep returned nothing.
6. Not used.
