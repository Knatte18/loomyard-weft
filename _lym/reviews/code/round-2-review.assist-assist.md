---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/beamcli/cli.go:503
    summary: The no-prime-claim flag has independent spellings instead of the single named constant required by PATTERN-code-conventions.
    fix: Declare exported NoPrimeClaimFlag in internal/beamcli/cli.go with value no-prime-claim and use it in BoolVar and MarkHidden. In internal/machinecli/start.go import beamcli, remove the independent noPrimeClaimFlag declaration, and construct the child argument as the -- prefix plus beamcli.NoPrimeClaimFlag. Use the owning constant for the flag lookup in internal/beamcli/start_test.go; keep literal expected child arguments in behavioral assertions.
  - id: F16
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/desktoplauncher/desktoplauncher.go:205
    summary: The new launcher lock bypasses the lock-path API and places never-tracked state outside .lym contrary to PATTERN-lym-files.
    fix: In internal/desktoplauncher/desktoplauncher.go declare a named launcherLockFileName constant for launchers.lock. Keep standalonestate.UserStateDir as the shared machine root, derive lockPath with lock.TrackedLockPath(stateDir, launcherLockFileName), create filepath.Dir(lockPath), and pass lockPath to lock.AcquireWriteLock. Keep the existing Add and Remove critical sections and update the lock-location comment to describe the resulting per-user .lym/locks path. Do not call standalonestate.Derive.
prior:
  - id: F1
    status: resolved
    answer: By-slug children carry the hidden flag through beamcli into orchcli preparation; beam-only EnsureLive calls do not claim the prime.
  - id: F2
    status: resolved
    answer: Top-level worktree entries carry name and line; hub lines render holds. Summary assertions cover attach commands and holds.
  - id: F3
    status: resolved
    answer: Failed and refused dobby outcomes now fail the worktree and verb while retaining warnings.
  - id: F4
    status: resolved
    answer: The extracted production parser has exact envelope and error-fallback assertions.
  - id: F5
    status: resolved
    answer: Negative owner rows retain an owned board so catch-up cannot mask an erroneous addition; the owned-beam foreign-prime row is present.
  - id: F6
    status: resolved
    answer: Tests compare workspace paths and terminal directories and titles, and cover entry, terminal, workspace and hub failures.
  - id: F7
    status: resolved
    answer: TestHubCatchUp compares Launcher against the fixture board directory and recorded shortname.
  - id: F8
    status: resolved
    answer: The planted-launcher removal test carries its testtiming keep marker.
  - id: F9
    status: resolved
    answer: Lookup failures now name machine start and its recovery command, with the original cause retained.
  - id: F10
    status: resolved
    answer: WSL launcher generation rejects double quotes, dollar signs, backquotes and backslashes before either write; tests cover those refusals.
  - id: F11
    status: resolved
    answer: Add and Remove hold the same exclusive lock throughout their critical sections. The new lock-placement violation is F16.
  - id: F12
    status: resolved
    answer: A table drives the real user-dirs file read, absent-file fallback and read-error branch.
  - id: F13
    status: resolved
    answer: The test-isolation background now states that XDG_DESKTOP_DIR points at the run directory on every platform.
  - id: F14
    status: resolved
    answer: The machine sandbox exclusion now describes start and its external effects alongside name.
---

### [BLOCKING] Declare the child flag once

**Location:** internal/beamcli/cli.go:503

**Issue:** Registration and hiding repeat the flag name,
and `internal/machinecli/start.go:42` independently spells its child-command form.
`pattern/PATTERN-code-conventions.md` requires one named constant referenced everywhere.
These production readers can drift independently even though the current spellings agree.
The round requires every listed constraint violation to be BLOCKING, regardless of its operational severity.

### [BLOCKING] Place the launcher lock through the lock-path API

**Location:** internal/desktoplauncher/desktoplauncher.go:205

**Issue:** The new lock is joined directly beneath `standalonestate.UserStateDir`.
`pattern/PATTERN-lym-files.md` puts never-tracked lock state under `.lym` and requires lock paths through `lock`.
The decision record exempts the desktop launcher files from hub placement, not their coordination lock.
`internal/lock/tracked.go` can derive the shared lock beneath the same per-user root without using `Derive`.
No commit in the review range carries a `Pattern-Departure:` trailer for this placement.

## Review basis

Reviewed the committed range `80ead58445718f329a7b068da7e40c3b7e11f883..HEAD` at `a9a3a9fa`.
The board entry, decision record, round focus and incremental changed-file list were read.
Both new findings sit on round-two changed hunks.
All four lens reports are saved beside this report.

The no-owner child path skips the preparation claim,
and the beam-only dobby start never enters the prime claim branch.
Worktree skip details and attach commands reach `output.Summary` through their top-level list and line fields.
The hub line carries the hold machine, subjects and local re-run.
Failed or refused dobby entries now produce a failed worktree and nonzero exit.
The tests cover the extracted child-envelope parser and the requested routing and failure cases.

## Verification limit

The foreground tier-zero gate was attempted for machinecli, beamcli, orchcli, desktoplauncher and dobbyengine.
It could not acquire a slot because the sandbox makes `_bolt/.lym/gate` read-only.
No tests ran in that attempt; the verdict rests on source review and the supplied gate report.
The blocked gate invocation is an environment limitation, not a finding against the diff.
