---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/orchengine/cleanup.go:44
    summary: a dry run of the orch handoff cleanup takes the state lock through a blocking acquire, which creates the lock file when it is absent
    fix: In cleanupHandoffs, when apply is false and os.Stat(p.StateLockPath) reports fs.ErrNotExist, skip the lock acquire and read the state without it (an absent lock file means no cycle holds it); keep the blocking lock.AcquireWriteLock for apply, and for a dry run whose lock file exists. Add a row to TestCleanupHandoffs that deletes the lock file before a dry run and asserts the file is still absent afterwards.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/battencli/cleanup.go:46
    summary: the CleanupReport.Applied comment says it reports whether the pass changed anything, but the field carries the apply flag
    fix: Reword the comment on CleanupReport.Applied to say it reports whether the pass ran with apply true, a dry run being false.
---

### [MEDIUM] Orch dry run can create the state lock file

**Location:** internal/orchengine/cleanup.go:44
**Issue:** The plan's shared decision `dry-run-shape` (cards 1, 13, 16, 18, 19, 20) says a dry run creates no lock file and removes nothing.
`cleanupHandoffs` calls `lock.AcquireWriteLock(p.StateLockPath)` in both modes, and the lock package's flock opens the path with create semantics.
When `p.Dir` exists and the state lock file does not, a dry run, including the `orch` segment of `lyx cleanup`, leaves a new `state.lock` behind.
`internal/battencli/cleanup.go` already meets the rule with `takeLock`, which stats the lock file first in a dry run.
No row of `TestCleanupHandoffs` asserts the lock file is absent after a dry run, so the gap is also uncovered.
Everything else in the focus list holds: the sweep probes an existing lock file only and never creates one, and the inherited-run-directory rule is in `resolveRootPath`.
The `tmuxkit.Main`, testtiming and gate paths remove their run directories, and card 6 drops `TMPDIR` for the inherited marker.
Every test package, `internal/testkit/tmuxkit` included, calls `Main` from a `TestMain`, and the renamed guard keeps its fixture rows.
The bare-temp scan landed with its PATTERN entry in the same commit, and the budget rows equal the top-level test counts.
`lyx gate test` over the touched packages is green.

### [NIT] CleanupReport.Applied comment misdescribes the field

**Location:** internal/battencli/cleanup.go:46
**Issue:** The comment reads "Applied reports whether the pass changed anything", but the field is set to the `apply` argument, so an apply that removed nothing still reports true.
The verb prints it as `"applied"`, meaning the mode.
