---
format: 5
approved: true
language: go
---

# Plan: ly-drive safe self-repairs instead of always handing back

A loom-launched ly-drive driver clears the stops a single re-step would clear, without the hub.
Go marks an infrastructure failure as transient at the producer or bootstrap boundary, and the step error envelope carries that class.
The landing producers return a transient push or GitHub failure as an error instead of a verdict, so the mark reaches it.
Every step records the build identity of the `lyx` that ran it, so `lyx shed status` can report `binary_changed` after a deploy.
The driver re-steps once for each, and hands back every other stop.
At a hand-back it parks in its own session behind a marker file, and `lyx loom start` resumes it by typing one line instead of spawning a fresh driver.
The Go driver (`lyx loom run`), `lyx loom approve`, verify triage and the reed mailbox are out of scope.

## Card Index

1 — shedengine-transient-mark — declare the transient mark in shedengine and persist it with `failed`
2 — gitexec-transport-classifier — gitexec classifies its own transport failures
3 — githubclient-transient-classifier — githubclient classifies its own transient API failures
4 — agent-start-not-started — surface a never-ready agent start on shuttle's Result and wrap `ErrNotStarted` in producer errors
5 — shedtransient-package — one package translating leaf classifications into the mark
6 — step-envelope-transient — every step error envelope carries `transient`
7 — transient-boundary-wiring — `NewShed` and both step bootstraps classify; Transient Stop Invariant
8 — producer-transport-errors — Publish and Finalize return transient push and GitHub failures as errors
9 — step-build-identity — record each step's build identity; status derives `binary_changed`
10 — shedrun-park-marker — declare the park marker's filename and path in shedrun
11 — driver-prompt-park — launch prompt parks at a hand-back
12 — driver-resume-line — compose the one line that resumes a parked driver
13 — loom-start-resume — `lyx loom start` resumes a parked, live driver through shuttle `Send`
14 — ly-drive-skill — the skill's closed re-step list, parking, binary watch, resume and records

## Shared Decisions

- **The classifier is a told `Shed` field, not a producer wrapper.**
  `shedengine.Shed` gains `Transient func(error) TransientClass`, which `shedbuild.NewShed` fills with `shedtransient.Class`.
  Wrapping every built producer would break `internal/loomrecipe/shape_test.go`'s concrete-type assertions on the `Publish` and `Finalize` rows, and `NewShed` is the one assembler both recipe sheds already share.
- **`internal/shedtransient` is the single translation site.**
  `shedengine` stays stdlib-only (Shed Producer-Seam Invariant), the leaf packages gain no `shedengine` import, and both boundaries (the `Shed` classifier and the `PreStep` hooks) call the same mapping.
- **A transient failure is an error, never a verdict.**
  The classifier sees only a producer's returned error, and the landing producers today turn every push and GitHub failure into `Stuck`.
  So a producer returns a failure `shedtransient.Class` classifies as a hard error, and keeps its verdict for every other failure.
  Loom's bootstrap makes no remote call today, so its `PreStep` mark is wired but inert.
- **A never-ready agent start is reported by shuttle, not inferred.**
  `RunGated` folds `ErrNotStarted` into a died or timeout result by design, so it reports `Result.NotStarted` and the producers that turn that result into an error wrap `ErrNotStarted`.
- **Build identities travel as parameters.**
  `newStepRecorder` and `lastStepOf` take the identity, so no test substitutes package-level state.
- **The park marker's name is Go-declared and skill-pinned.**
  `shedrun.ParkMarkerFileName` is the only spelling in Go, and a `loomcli` test pins the skill to it.
- **The resume waits only on a not-ready pane.**
  `shuttleengine` marks its not-ready refusal with `ErrPaneNotReady`, and `lyx loom start` retries only that, since re-sending after any other `Send` error could type the resume line twice.

## verify:

```
go build ./...
go vet ./...
go test ./...
go test -tags integration ./internal/shedengine/... ./internal/gitexec/... ./internal/githubclient/... ./internal/shuttleengine/... ./internal/burlerengine/... ./internal/shedadapters/... ./internal/shedtransient/... ./internal/shedbuild/... ./internal/landingshed/... ./internal/shedverbs/... ./internal/shedrun/... ./internal/loomcli/... ./internal/battencli/...
go vet -tags smoke ./internal/shedengine/... ./internal/gitexec/... ./internal/githubclient/... ./internal/shuttleengine/... ./internal/burlerengine/... ./internal/shedadapters/... ./internal/shedtransient/... ./internal/shedbuild/... ./internal/landingshed/... ./internal/shedverbs/... ./internal/shedrun/... ./internal/loomcli/... ./internal/battencli/...
```
