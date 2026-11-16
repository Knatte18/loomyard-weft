# Merge main into the beam adopt-build

Brings `main` (up to 194650f0) into the beam `adopt-build` as one merge commit, 4153c5ef.
Three paths conflicted, and two more files needed changes because the beam used a constant that `main` deleted.

## Conflict resolutions

- `internal/orchcli/cli.go`: kept both sides' new `orchCLI` fields and their wiring.
  That is the beam's `registerOrch`, `orchRegistered`, `endOrch` and `endSubject`, and `main`'s `machineName` and `injectRemoteControl`.
  Dropped `endTestMap` and its wiring, since `main` removed the coverage-map upkeep pass and its `runproc.RoleTestMap`.
- `internal/orchcli/start.go`: the resume path keeps the beam's `resumeOrch`, which ends this worktree's own earlier orch process and resumes again.
  `resumeOrch` now also takes `main`'s `remoteControlName` and passes it to `orchSpec`,
  so a resumed session gets remote control as `main` intended.
- `internal/runproc/doc.go`: the hub roles list follows `main` (no `RoleTestMap`), and the beam's line on the prime's `RoleOrch` registration stays.

## Fixes the merge needed outside the conflicts

`main` deleted `runproc.RoleTestMap`, but the beam's `EndSubject` added a reference to it after the merge base:

- `internal/runproc/end.go`: removed `RoleTestMap` from `subjectPassSpared`, and removed the coverage-map pass from `EndSubject`'s doc comment.
- `internal/runproc/doc.go`: the same doc fix for the `EndSubject` paragraph.
- `internal/runproc/end_test.go`: removed the `RoleTestMap` record from the `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` fixture.
  No row expected that record to be signalled or ended.

These fixes are part of the merge commit, not separate commits after it.

## Checks

`go build ./cmd/lym` and `go vet` pass on `orchcli`, `runproc` and every other package both sides changed.
The gate has not run yet.
