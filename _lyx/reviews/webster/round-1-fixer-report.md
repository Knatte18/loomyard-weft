# Round 1 fixer report

## Fixed

- F1: the Bouncer's halt on a malformed escalation record now names the file and the way forward (`lyx loom start` after fixing or deleting it) in `internal/shedadapters/bouncer.go`.
  `contracts/specs/refusal-spec.md` gains the `escalation record unreadable` row in the shed section.
  `TestBouncer_Escalation_MalformedRecordHaltsStuckNamingTheWayForward` reaches the halt untagged.
  `go test ./internal/shedadapters/ ./contracts/...` passes.

## Deferred

None.
