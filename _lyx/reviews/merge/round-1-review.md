---
verdict: APPROVED
---

No finding survives consolidation.
The focus directive's five questions are each settled below.

## Settled focus questions

- **Registration before the start check.**
  `registerRunProcess` runs first in the `drive` RunE (`internal/loomcli/drivewiring.go:70`), before the driver lock and before `sheddriver.Run`.
  A start-check refusal or check error hands back through `handBackAs` or `handBackFailure` and the process exits.
  No process removes its own record, and a dead pid's record is pruned on the next read, so a refused start leaves nothing unreleased.
  `StartCheck` is `c.finalizeGuard()`, and `c.parentCodeDirt` is set at wiring (`internal/loomcli/cli.go:190`), so the seam is never nil there.
- **Board CLI.**
  The parent's selector groups (`payloadKeys.exclusive` as `[][]string`) and the task's deferrable `sync` flag sit in separate command blocks.
  `sync` takes no payload, so no selector applies to it.
  `boardengine.DeferrableSyncFlag` is read in `internal/boardcli/cli.go`, `internal/boardengine/spawn.go` and `cmd/lyx/main.go`, and `cmd/lyx/main_test.go` still asserts it.
- **Config layers.**
  `internal/runproc` imports none of `runprofile`, `configengine` or `yamlengine`, and the task's range touches none of them, so no task code reads a profile key through a path the parent changed.
- **Docs.**
  The merged prose of `boardcli`, `boardengine`, `fabricengine`, `loomcli`, `sheddriver` and `docs/overview.md` states one contract.
  The parent's `StartCheck` text and the task's `RestepRecorded` and registration text sit in different paragraphs without contradiction.
- **Test economy.**
  The parent's `StartCheck` rows in `internal/sheddriver/drive_test.go` and the task's registration and deferral tests pin different behaviors.
  No merged test duplicates another.

## Rejected

- Nothing pins that the `drive` RunE calls `registerRunProcess` (code-tests, LOW): the gap lives in the task's own change, which the Webster review covered, and the interaction added by the merge, a hand-back after registration, leaves no state because the record self-prunes.
- A refused second driver still registers before the lock refuses it (code-correctness, out of lens): the task's own design, harmless because the record is pruned once the process dies.
