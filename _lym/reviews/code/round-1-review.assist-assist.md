---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/reedengine/lifecycle.go:882
    summary: Stored orch resume commands retain the previous implicit remote-control enablement.
    fix: Make orch revival resolve the current remote-control configuration before replaying a stored command, covering both Resume in internal/reedengine/lifecycle.go and the absent-pane reopen in internal/reedengine/strand.go through a provider-neutral integration with orch and shuttle. Regenerate the provider command through its owning engine rather than parsing provider flags in reed. Preserve explicit opt-ins and the RC button behavior, and cover revival of a strand created under the old implicit default.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:19
    summary: The summary misstates absent-file reconciliation and beam orch launch behavior.
    fix: Correct the absent-file paragraph to describe ReconcileHubWideAt seeding the base before reinterpretation, including a notice for a seeded header-only file. Replace the claim that beam orch never sets RemoteControlName with the fact that beam orch uses the same configured launch gate through BeamUp.
---

### [MEDIUM] Stored resumes retain the old implicit opt-in

**Location:** internal/reedengine/lifecycle.go:882

**Issue:** Goal/Fix bullets 1 and 4 require launches and resumes to remain off without explicit configuration.
An orch launched under the previous default stores a command containing `--remote-control`.
`internal/shuttleengine/claudeengine/command.go:595` constructs that stored argument.
Reed replays it at `internal/reedengine/lifecycle.go:882` without resolving the new config.
The absent-pane reopen at `internal/reedengine/strand.go:615` has the same behavior.
A server restart can therefore start a new provider process with remote control on despite no operator opt-in.
The landing summary acknowledges this exception, but the decision record contains no amendment accepting it.
The affected-set lens independently identified the same defect; it is consolidated here once.

### [LOW] Landing claims disagree with their callers

**Location:** _lym/landing/summary.md:19

**Issue:** The summary says an absent orch.yaml gets only a marker.
`internal/hubreconcile/walk.go:224` first reconciles with `SeedDefaults: true`.
`internal/configsync/configsync.go:430` seeds an absent per-worktree base from the prime or a sparse header.
The later reinterpretation flags that header-only file, so the walk can notify the orch too.
The summary also says beam orch does not set the remote-control name.
`internal/beamcli/cli.go:540` calls `orchcli.BeamUp`, which uses the same gated specification as the prime orch.

## Rejected

- The tests lens proposes a separate assertion on the registry binding and persistent ID.
  The callback behavior and generic marker/notification lifecycle already have behavioral tests;
  the reported gap is a literal registration assertion, with no distinct incorrect behavior demonstrated in this binding.

## Review evidence

Reviewed commits from `a042662792534f728cc1069f23d9830755b71af8` through `5ccddabc294f25a0ac09c5e59843215cd7d13936`.
Empty, null and commented-only YAML leave the decoded key map empty; lookup is safe and produces a notice.
Explicit keys remain unchanged and unflagged, and parse failures return an error before any write.
Nil `rewrote` retains flagged notices in the reconciliation loop; markers provide once-per-hub application.
The registry ID is unique, and the template change changes the registry fingerprint.
No declared PATTERN amendment or additional mandated document was identified.
The loader tests use literal expectations and preserve the explicit-on empty-name refusal.
The Root-Cause trailer names the template defect the diff changes.

The supplied reach, redundancy and mutation reports were considered as evidence, not rerun results.
An attempted tier-0 gate for orchengine and configreg reported a read-only run-record lock path.
It returned no test result before cancellation; this review makes no claim that the attempted run passed.
