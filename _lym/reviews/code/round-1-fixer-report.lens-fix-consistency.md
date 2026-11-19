```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the change set against its surroundings and found no findings.

- **Old names:** no reference to `BootTime`, `StartedAt` or `started_at` for the loop's pid record is left in the code or the package docs.
- **Docs and tests:** `doc.go` (pid-file fields, early-end sources, unmarked-exit and reconciliation paragraphs), the `runLoop` doc and the `LoopSpec.BootID` doc all describe the same behaviour as the code. Every `loopExitEvidence` caller passes a boot id string, and the pid-record writers in `pairteardown` and `loomcli` set no `StartedAt`.
- **Panic tests:** both panic-with-child tests are back to the single panicking `Now`, and no stale first-read comment remains anywhere in `internal`.
- **`LockGrace`:** its doc ("a few seconds") agrees with `loopLockGrace` being 5s, and zero keeps that default.

Tool calls (2 of 6):
1. `grep` for stale `BootTime`, `started_at`, `LockGrace` and reconciliation text across `internal` and `docs`. My context did not hold the files the fixer did not read.
2. `grep` for leftover `clockReads` and first-read comments, and for every `loopExitEvidence(` and `LoopPIDRecord{` caller. My context could not show callers outside the files the fixer read.
