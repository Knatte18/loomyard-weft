```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set for round 1. For each changed signature, output or behaviour in `9447bbda..HEAD`, I list what reads or asserts it and whether the diff covers it.

**Per change, the set found:**

- **`lym hibernate watch` verb and `spawnDaemon`, `killDaemon` and `watch.log` removed.** Covered.
  - A repo-wide grep for `hibernate watch`, `watch.log` and `hibernate.*daemon|watch` finds no stale reference to the removed verb or its log.
  - The remaining `watch.log` hits belong to `orchcli` and `orchengine`, which is a different daemon.
  - `fabriccli/anchor_test.go:410` keeps only a test name ("TheHibernateDaemonIsLiveForThePrime"). The test is behaviour-correct, so no finding.
  - `hibernateengine/watch.go` is still a listed file in `cmd/lym/daemonwakeups_test.go:36`.
  - `pattern/PATTERN-daemon-wakeups.md:14` was updated to name it as run in the prime's dobby, with the same 1 s base and 30 s ceiling as `triggerPollBase` and `triggerPollCeiling` in `watch.go`.
  - `shellguard/rows.go:170` still points agents to `lym hibernate request`, which still exists.
- **`Deps.Spawn` and `Deps.Kill` replaced by `DobbyLive`; `RequestPath` and `NewDobbyCarrier` added.** Covered.
  - Callers: `beamcli/dobbydeps.go` and `fabriccli/anchordeps.go` (the request JSON's `Phase` field plus `hibernateEndedPhases`).
  - Tests: `beamcli/dobbydeps_integration_test.go`, `beamcli/dobby_test.go` (`TestHoldReexec`) and `fabriccli/hibernatedaemon_test.go`.
  - The ended-phase list is pinned to `hibernateengine.TerminalPhases()` by `TestHibernateEndedPhases_AreTheEnginesTerminalPhases`.
  - Docs updated: `beamcli/doc.go`, `fabriccli/doc.go`, `hibernatecli/doc.go` and `docs/overview.md`.
- **`dobbyWatchDirs` gained a `hibernateRequestPath` parameter.** Covered. The two callers, `dobbydeps.go` and `dobbypairanchor_test.go`, are both updated, and both the prime and beam cases are asserted.
- **`CaptureRequest` and `CompareRequest` gained `WorktreeDir`; `stepRun` gained `WorktreeDir` and `Name`.** Covered.
  - Every production construction site sets `WorktreeDir`: `driftcli/wiring.go:121` and `:137`, and `loomcli/driftwiring.go:200` and `:215`. `driftcli/cli.go` goes through those helpers.
  - The `stepRun` literals are only in `capture.go` (twice) and `compare.go`. All three set `WorktreeDir` and `Name`.
- **`BaselineInputs.AnchorDir` added and required.** Covered. The only production caller, `loomcli/driftwiring.go:126`, sets it. The required-field row and the `capture_command` rows are in `seats_test.go`.
- **`CommandStep.Dir` no longer defaults to `"."`; an unset dir now means the anchor, and relative means the worktree root.** Covered.
  - Docs updated: `driftengine/doc.go`, `driftcli/doc.go`, and the field comments in `config.go`, `capture.go` and `compare.go`.
  - Tests that still use `Dir: "."` (`baseline_test.go:89`, `seats_test.go:156`, `config_test.go:384`, `verify_test.go:148` and `:822`) construct steps directly. `"."` still resolves to the worktree root, so they stay valid.
  - `CheckDirs` is wired from `configcli/hubwide.go`.
- **`gateslot.ResolveCommandDir`, `StatCommandDir` and `CheckDirs` added.** Covered. They are used by `driftengine` (`steprun.go`, `seats.go`, `config.go`), and `gateslot/config_test.go` tests `ResolveCommandDir`.
- **`PATTERN.md` and `pattern/` declared changes.** The directive declares none, and the only `pattern/` change in the range is the `PATTERN-daemon-wakeups.md` line above.
- **Commit trailers.** `git log` over the range shows no `Pattern-Departure:` trailer.

**Findings:** none from this lens.

**Tool calls (1 of 6):**
1. One `grep` batch over `internal`, `cmd`, `docs`, `pattern`, `contracts` and `PATTERN.md`, covering the `CaptureRequest`, `CompareRequest`, `stepRun` and `BaselineInputs` construction sites, stale hibernate daemon and watch references, and `Dir: "."` usages. My context had the diffs but not the repo-wide callers, docs or tests outside the range.
