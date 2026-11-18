---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `cgroupMemoryMaxFile` moved to `internal/proc/memorylimit.go`, and `readMemoryLevel` uses it.
- F2: the gate's cap test runs `head -c 268435456 /dev/zero | tail`, with the background `sleep` and its two comment lines gone.
- F3: `lym gate test --tier 1 ./internal/proc -- -count=50 -run TestRunMemoryCapped` (run `20261118T142443Z-tier1-444250`, 15:24:25 to 15:24:43)
  ran beside a tier-1 run of `./internal/gatecli` (run `20261118T142527Z-tier1-444438`, 15:24:27 to 15:25:27).
  The spans overlap, and `_lym/landing/summary.md` now names both runs, their spans and 950 results.
- F4: `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` polls for the scope's removal.
- F5: the holder starts under `setsid`, and the `scopeStartScript` doc comment says it leaves the command's process group.
- F6: `TestRunMemoryCapped` gains a row with a cap of `testMemoryCap + 1`.
- F7: `TestRunMemoryCapped_TheHolderIsNeverAChildOfTheCommand` lists the command's children.
- Self-check changes:
  - `scopeStartScript` exits 125 when `setsid` is not on the scope's PATH, with `TestRunMemoryCapped_AScopeWithoutSetsidIsAnErrorAndNeverRunsTheCommand`.
  - The F7 test skips on a kernel with no `/proc/<pid>/task/<tid>/children`.
  - The stale dash sentence is gone from the `scopeStartScript` comment.
  - The `memorycap_linux_test.go` header names the holder tests.
  - `summary.md` names the `setsid` holder and the new tests.
- Commits: `6cd65f7c` (F1, F5), `cfef55a3` (F2), `9e5462d7` (F4, F6, F7) and one for the self-check changes.
  F1 and F5 share `memorycap_linux.go` and F4, F6 and F7 share the test files, so each pair went in one commit.
  `_lym/landing/summary.md` (F3) sits behind the `_lym` link and is not committed.
- `lym gate test --tier 1 ./internal/proc` and `./internal/gatecli` pass.
- Gate repair: `TestGofmt_ModuleFilesAreFormatted` failed on `internal/proc/memorycap_linux_test.go:223`, where `cmd :=exec.Command` lacked a space.
  The space is restored in its own commit, and `lym gate test --tier 0 ./cmd/lym` passes.

## Self-check

- fix-consistency, LOW, summary holder bullet and Tests list stale: fixed.
- fix-consistency, LOW, `memorycap_linux_test.go` header omits the holder tests: fixed.
- fix-proposal, MEDIUM, a holder with no `setsid` fails silently: fixed with the exit-125 guard and its test.
- fix-code-correctness, LOW, the same missing `setsid`: merged into the finding above.
- fix-proposal, LOW, the F7 test fails without `CONFIG_PROC_CHILDREN`: fixed with a skip.
  fix-code-correctness raised it too, and it is merged.
- fix-code-correctness, LOW, stale dash comment: fixed.
- fix-proposal, LOW, the F4 test cannot tell a holder that never started: disputed, premise false.
  `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap`'s row `AStartThatReturnsOnceTheCommandHasEnded` (`memorycap_linux_test.go`) reads the kill after the command ended, which fails with no live holder.
  The `setsid` guard now stops the holder from failing silently.
- fix-affected-set: no findings.

## Disputed

- Self-check fix-proposal LOW on the F4 test: ground `premise false`.
  The holder's liveness after the command ends is pinned by the tree-kill row named above, so the F4 test only owes the holder's end.

## Deferred

None.

## Rulings

None.
