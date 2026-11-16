# Merge main into the beam adopt-build

Brings the parent branch `main` into the beam `adopt-build` as a true merge.
The merge carries main's changes since the beam last merged it; it adds no change of its own beyond the fixes below.

The incoming work, as a reviewer will see it in the diff:

- `orchcli` and `orchengine`: orch sessions turn on remote control themselves, switchable per hub.
- `gatecli`, `gateslot`, `dobbyengine`, `testcover`, `impactset`: the test-map build is reworked, and the coverage-scope and yield code is removed.
- `shedarbiter`, `loomcli`, `loomengine`, `contracts/stencils`: arbiter calls gain a fix path with its prompt fragments.
- `reedengine`, `shuttleengine`, `claudeengine`, `boardengine`, `creelcli`: smaller feature and doc changes.
- `docs/research`: new research reports and captured data for the test-map work.

Two files conflicted and were settled by hand, and two follow-up commits fix what the merge left behind:

- `internal/orchcli/start.go`: the conflict resolution keeps both sides' changes; a follow-up reflows the wrapped `resumeOrch` doc comment at sentence boundaries.
- `internal/runproc/end_test.go`: the `HubOwner` subtest name still referred to `TestMap`, which main removed; the name now drops it.

No issue is closed by this task.

## To check

- Read the conflict resolutions in `internal/orchcli/start.go` and `internal/runproc/end_test.go`.
  They are the only merge content that is not main's own commits.
- No regression check ran for this task, and the gate was not re-run by hand after the last fix.
