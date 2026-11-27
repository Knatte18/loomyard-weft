# Give transient halts their own anomaly kind and log parent-lock waits

A Finalize step that failed on a DNS lookup of `api.github.com` was classed `transient=github-api`,
yet its halt note read `producer-hard-failure`.
Reflection took an outage that a second step cleared for a defect signal.

- A failed halt whose status records a transient class now writes `anomaly: transient-stop` in its halt note.
  `loomengine.ClassifyHalt` takes a new `transient` argument for this.
- The reflection template drops a `transient-stop` halt.
  It files one only when the notes show such halts at one row that the re-steps never cleared,
  or a failure classed transient that no retry could fix.
- The parent-lock wait in `landingshed` now logs its start, each change of holder and its end.
  The end line says whether the lock was granted, with both the wall-clock time (`waited`) and the monotonic time (`awake`).
  A gap between the two points to a wall-clock step, or to a host suspend where the monotonic clock pauses in suspend.
  This is there to explain long waits:
  the failed step spent about nine hours between its parent-lock wait mark and its refusal,
  and the trace did not show whether the lock was held that long or the host was suspended.

Closes #269

## Design decisions

- **The persisted state stays `failed`.**
  The issue asks for a retryable stop rather than `failed` / `producer-hard-failure`.
  This change adds no new run state and leaves the status write as it was:
  the status already records the transient class beside `failed`, and the re-steps key on that class.
  The halt note still says `state: failed`; only its anomaly kind differs.
- **A halt note is still written**, under its own kind, so the event stays on disk with its trace file.
- **The kind covers every transient class**, including `agent-start` and `host-resources`, not only network failures.
  A step loop re-steps each once.
  The Go driver also re-steps `git-transport` and `github-api` with waits and sends the other classes to the tackler.
- **Precedence among failed halts:** build skew wins over transient, and an interrupted stop wins over both.

## To check by hand

- A failure with a transient class is still persisted as state `failed`, not as a separate retryable stop
  (the failed-step persist in `internal/shedengine/run.go`).
  Confirm that `failed` plus the recorded class is enough for every reader of the status.
- Confirm that the `transient-stop` texts hold for every transient class:
  that a re-step can clear the failure, and what the step loop, the Go driver and the prompt-driven driver each do with it.
  The texts are the doc comment on `AnomalyTransientStop` in `internal/loomengine/anomaly.go`,
  the package doc in `internal/loomcli/doc.go`,
  and the drop rule in `contracts/stencils/friction/friction-template-reflection.md`.
- The new parent-lock log lines have no test.

## Merge danger

**Door:** two-way
**Blast radius:** loom
