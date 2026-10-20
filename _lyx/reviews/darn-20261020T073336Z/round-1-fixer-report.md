# Fixer-report, darn round 1

## Changes

- F1 (MEDIUM): `internal/shuttleengine/gate_test.go` has a new row in `TestAttach_GateThreading` that attaches to a live run through `AttachIfLiveGated`.
  Fixed differently: the review proposed a failing gate that forces a re-prompt, and the row uses a passing gate instead.
  A passing gate fails the test just as well if `attachAndWait` drops the gate (`Result.Gate` is nil and the gate is never called), and needs no re-prompt machinery.
  Commits `fix F1` and, for the test's doc comment naming the new entry point, a second `fix F1` commit.
- F2 (LOW): `agentFake` in `internal/sheddriver/tackler_test.go` records the gate handed to `Start` and `AttachIfLive`.
  `TestTacklerCall_SpawnPath` asserts every recorded gate is empty, which covers the start rows and the re-attach row.
- Verify gate repair: `TestGofmt_ModuleFilesAreFormatted` failed on `internal/sheddriver/tackler_test.go`, where the new `gates` field in `agentFake` left the `startErr` field block misaligned.
  `lyx loom fmt` realigned it in a follow-up `fix F2` commit.
- F3 (NIT): the `AttachIfLiveGated` doc comment in `internal/shuttleengine/attach.go` stands alone.

`lyx gate test ./internal/shuttleengine ./internal/sheddriver` passed.
Nothing was pushed and no `_lyx` path was touched by git.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the `TestAttach_GateThreading` doc comment and `testtiming:keep` line omitted `AttachIfLiveGated`; fixed as proposed.
- fix-proposal: LOW, the same stale doc comment; duplicate of the fix-consistency finding, fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
