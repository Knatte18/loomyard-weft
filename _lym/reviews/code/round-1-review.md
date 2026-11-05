---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/wire.go:698
    summary: "A fixer pane-process record counts as a live wait for as long as the pane process runs, so a fixer strand left behind after a non-done finish can hold the quiet notice indefinitely."
    fix: "In `gateProcessLive` in internal/battencli/wire.go, count a fixer-role record only while it is plausibly working: skip a record whose `RegisteredAt` is older than the fixer run's own timeout bound (read it from the same gatefixer or shuttle config the fixer is spawned with, never a literal), and keep counting `runproc.RoleGate` records as now. Add a row to `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` that registers this process as a `gatefixer.RoleGateFixer` with an old `RegisteredAt` and expects waitLive false. Also correct the claim in `_lym/landing/summary.md` and the commit message that a hung fixer delays the notice by its timeout at most, if the bound is not enforced."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire_seams_test.go:546
    summary: No test row proves a gate or fixer registered under another run's slug leaves this run's notice alone, so a wrong owner key in `gateProcessLive` would pass every test.
    fix: "In `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`, before the added registration loop (which leaves a live record under this run's slug), register `runproc.SelfRecord(runproc.RoleGate)` under a different owner name with `runproc.Register(runProcDir, \"another-slug\", ...)` and assert `readAgentActivity` returns waitLive false with a nil error. Then remove that record by its literal path or leave it, since the later rows register under this run's slug and expect true."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/wait_test.go:445
    summary: The run-end stamp is asserted only in the cleaned-up done row, so a regression that stamps only on done would pass although doc.go and the commit message say every finalize stamps.
    fix: "In `TestRun_Wait_Classification` in internal/shuttleengine/wait_test.go, assert for every row, outside the cleanup switch, that `filepath.Join(filepath.Dir(runDir), runEndStampFileName)` exists and its modification time is not before `result.EndedAt` minus one second. Keep the existing `ReadNewestRunEnd` assertion in the cleaned-up done case."
---

### [MEDIUM] A lingering fixer pane process can hold the quiet notice forever

**Location:** internal/battencli/wire.go:698
**Issue:** `gateProcessLive` counts any live record of a fixer role under the pair's slug, and `gatefixer.Registrar` (internal/gatefixer/registrar.go:28) records the strand's pane process.
`List` keeps such a record for as long as that process runs.
Nothing in internal/gatefixer removes the strand when the fixer ends, and shuttle's `finalize` removes it only for a done run that does not keep its pane (internal/shuttleengine/wait.go:1954), plus a died own-window strand.
A fixer that times out or errors, or a window-0 strand reed leaves, can keep its pane process alive, and the run then never gets a quiet notice.
The change description says a hung fixer delays the notice by the fixer timeout at most, and I found no bound that ends a lingering pane.
The decision record asks that a notice fire when none of the sources shows activity for the threshold; an idle leftover pane is not activity.
Whether callers always remove the fixer strand on every outcome is unverified.

### [MEDIUM] No row for another run's gate or fixer registration

**Location:** internal/battencli/wire_seams_test.go:546
**Issue:** The focus asks that another run's gate cannot hold this run's notice.
Every added row registers under `taskLocation.WorktreeName`.
Replacing `runproc.List(dir, WorktreeName)` with an all-owner read or a wrong owner would pass all rows.
The behavior is added by the diff and has no covering test.

### [LOW] Stamp pinned for the done-cleanup outcome only

**Location:** internal/shuttleengine/wait_test.go:445
**Issue:** The stamp assertion runs in the cleaned-up done row, the one case where no run record can stand in for the stamp.
A change that stamped only on done would pass; died, timeout and api-error runs are covered by their record's mtime today, but the stated contract is that every finalize stamps.

## Rejected

- Code-correctness finding 2 (an unreadable runproc directory blocks every notice): the sibling reads in `readAgentActivity` (`verifytree.ReadMarker`, `parentlock.ReadWait`, `shuttleengine.ReadWaitMarkers`) propagate their errors the same way, so the new read follows the function's own convention.
- Code-tests finding 3 (no row for stamp write or read failure): both paths are a local-filesystem error that only falls back to the record reading, and a row for each would test a swallowed error rather than a behavior of the change.
- Code-tests finding 4 (no dead or recycled pid row in `gateProcessLive`): pruning of dead and recycled pids is `runproc.List`'s own behavior, which that package's tests pin; a row here would duplicate them.
- Code-affected-set returned no defects; the handler's own reading of the stamp's staleness (a stale stamp is an old timestamp, so the threshold still fires), the owner key (`originSlug` and the fixer registrars all use the worktree name `gateProcessLive` reads) and the threshold logic in `judgeAgents` (unchanged) found none either.
