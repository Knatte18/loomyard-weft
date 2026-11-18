---
dispositions:
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

## Changes

Commit `9571033b`, `proc: fix F8, F9`, covers both findings, because both edit `internal/proc/memorycap_linux_test.go`.

- F8: added a `//testtiming:keep` line with the reviewer's reason above `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` and above `TestRunMemoryCapped_AScopeWithoutSetsidIsAnErrorAndNeverRunsTheCommand`. The test bodies are unchanged.
- F9: added `SelfCgroupFile` and `CgroupMount` aliases, each with a doc comment, to `internal/proc/export_linux_test.go`. The holder test now passes `proc.SelfCgroupFile` to `cat` and `proc.CgroupMount` to `filepath.Join`.

`go vet ./internal/proc` and `lym loom fmt` were clean.

## Self-check

All four forks returned a report; each answered "no findings".

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings. The `cgroupRoot` literal in `cmd/testtiming/resources.go` is a separate package's own constant, outside the review's finding.

## Disputed

None.

## Deferred

None.

## Rulings

None.
