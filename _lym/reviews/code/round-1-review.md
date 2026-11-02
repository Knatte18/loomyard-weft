---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/pawlwiring.go:99
    summary: No test pins that the Beam-Up step surfaces the report's warnings or that a failed notice post never halts the row, which is fix 2 of the task
    fix: In internal/loomcli/pawlwiring.go give surfaceBeamUpWarnings (or the Beam-Up step closure) an injectable notify function, defaulting to orchcli.NotifyOrchAt with pawlProgram in pawlDeps. In internal/loomcli/pawlwiring_test.go add one table-driven test with rows for no warnings (nothing posted), warnings (one post carrying beamUpWarningLine's text) and a post that returns an error (the call still returns normally, so the row is not halted).
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/start_test.go:291
    summary: No row of TestStart_EnsuresTheRunsStrandBeforeTheOrch holds a runs strand and a loom-status strand together, so the precedence of the runs strand in ensureRunsStrand's loop is unpinned
    fix: In internal/orchcli/start_test.go add two rows to TestStart_EnsuresTheRunsStrandBeforeTheOrch, one with strands listed loom-status then runs and one with runs then loom-status, each with the runs strand stale (not live) and expecting a replace of the runs strand's GUID.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/sharedbootstrap.go:279
    summary: The call site that passes the session's strands into statusStrandAddSpec is unpinned, since the smoke tests pass nil and reverting the argument fails nothing
    fix: In internal/loomcli/smoke_drivingsurface_test.go add a row (or extend TestSmokeStatusStrandAcrossDriverSeeds) that adds a runs strand to a real reed session, runs ensureStatusStrand, and asserts the status strand was added in its own window beside the runs strand.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:539
    summary: docs/overview.md still says the loom status strand is always the session's overview window and that beam start only ensures the runs strand, which the diff makes untrue for a beam's session
    fix: In docs/overview.md lines 539 and 811 say the status strand takes a window of its own, never taking the viewer, when a beam's runs strand holds the overview. In lines 421 and 677 add that ensuring the runs strand replaces a loom-status strand and that a failure to add, replace or record it is a warning on the bring-up report. Follow the wording of the updated internal/loomcli/doc.go and internal/orchcli/doc.go.
---

### [MEDIUM] Beam-Up warning surfacing has no test

**Location:** internal/loomcli/pawlwiring.go:99
**Issue:** The task's fix 2 is that Beam-Up stops dropping `BeamUp`'s report. Only the pure `beamUpWarningLine` is tested. Reverting the step to `_, err := orchcli.BeamUp(...)` fails no test, nor does a `surfaceBeamUpWarnings` that returns the post error. `NotifyOrchAt` is called directly with the location, so the test has no seam to reach.

### [LOW] Runs-strand precedence over loom-status is unpinned

**Location:** internal/orchcli/start_test.go:291
**Issue:** `ensureRunsStrand` matches both roles in one `switch`, so a `runs` entry after a `loom-status` entry must still land in `runs`. The code does this correctly, but every test row holds one role only, so a reordered switch passes all of them.

### [LOW] Strands argument at the status strand call site is unpinned

**Location:** internal/loomcli/sharedbootstrap.go:279
**Issue:** `ensureStatusStrand` passes `statusResult.Strands` to `statusStrandAddSpec`, which is what sends the status strand to its own window in a beam's session. `TestStatusStrandAddSpec` pins the function, and the smoke tests pass `nil`, so dropping the argument fails nothing.

### [LOW] docs/overview.md describes the old overview-slot behaviour

**Location:** docs/overview.md:539
**Issue:** Lines 539 and 811 call the loom status strand the session's overview window. Lines 421 and 677 say `lym beam start` ensures the `runs` strand and nothing of the replace or the warning. The same diff updates the package docs for both, so the long-form doc is the one left behind.

## Rejected

- The loom-status replace-failure warning branch and the other `ensureRunsStrand` warning branches have no row of their own (lens:code-tests). The text is built by one shared `runsStrandWarning`, the add-failure row pins that path end to end, and the other branches only change the `what` string; a row per branch would duplicate the same coverage.
- The report's warnings are not surfaced when `BeamUp` returns an error (lens:code-correctness, noted only). Unchanged by the diff, and an error already halts the row as a Stuck.

## Handler notes

- Candidate 2 is confirmed from the code, so fix 1 is licensed. `internal/battencli/wire.go:1286` spawns `lym loom start --no-attach` in the child's anchor, the commit message and `_lym/landing/summary.md` both say so, and the summary's `wire.go` citation matches.
- The `runsStrandName` and `"loom-status"` renames leave no production literal behind. The remaining `"loom-status"` and `"runs"` strings are test fixtures and unrelated JSON keys.
- No `PATTERN.md` entry is violated and the diff declares no PATTERN change. No `Pattern-Departure:` trailer is present or needed.
- `PATTERN-dobby-orch-pair` has no `pattern/` background file, so none was read.
