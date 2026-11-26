```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that would fail if it broke. One behaviour has no such test.

**Per behaviour, the test that pins it**

1. The restore runs before the Go-driver spawn, after the pusher. Pinned by the `TestResumeVerb` row "a go-seeded blocked run with no strand spawns a Go driver once", which expects the events in the order `pusher`, `pane`, `wake` (`internal/loomcli/resume_test.go:272-273`).
2. The restore runs before the resume line is typed into a parked driver. Pinned by the row "paused parked driver is woken", with the same order (`resume_test.go:257-258`).
3. The restore runs once only when a driver is spawned or a resume line is attempted, never on a refusal or no-op. Pinned by the per-row `paneRestores` count, which every `TestResumeVerb` row checks (`resume_test.go:395-401`). The expected count comes from the row's own fields (`wantSpawns`, `wantSent`, `sendErr`), not from the code under test, so it is not tautological.
4. A failed restore still spawns the driver and returns a success envelope whose `status_pane` names the error and `lym loom start`. Pinned by the new row "a failed status-pane restore still spawns the Go driver and names start" (`resume_test.go:274-275`), together with the `wantPaneIn` and empty-`status_pane` checks (`resume_test.go:402-409`).
5. The success envelope carries no `status_pane` when the restore succeeds. Pinned by the `env.StatusPane != ""` check on every row (`resume_test.go:402`).
6. The `logger.Warn` call on a failed restore (`resume.go:199`) has no test. It is a log line, so this is no finding.
7. The `wire` closure, `reedEngine.Up()` followed by `ensureStatusStrand` (`wiring.go:1384-1389`), has no test. See F1.
8. A refused resume still brings no reed session up. Pinned by `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` (`smoke_resumearming_test.go:51-52`); only its wording changed.
9. The changed subtest "loom resume leaves a follow-off session off" (`smoke_drivingsurface_test.go:164-187`) still checks what its name says: `@lym_follow` stays `off`, and the window is unchanged from the shell window. The `Loosened-Check` trailer on c4f96e75a gives a measured cause (`@3` against `@1`), so this is no finding.

**Findings**

- **F1. MEDIUM, design.** Location: `internal/loomcli/wiring.go:1384`, the `restoreStatusPane` closure.
  - No test checks that a resume over a dead, stale or missing status strand brings a live one back.
    - `TestResumeVerb` replaces the seam with a counting fake (`resume_test.go:302-307`).
    - The only smoke test that resumes on the wake path checks the follow state, the window and the spawn message, never the status strand (`smoke_drivingsurface_test.go:175-186`).
    - The other tests that run `loom resume` are refusal paths (`smoke_resumearming_test.go:36`, `smoke_prebootstrap_integration_test.go:192,211`).
  - The mutation report agrees: dropping the closure's assignment at `wiring.go:1384` survives.
  - That restore is what the board entry asks for.
  - A test can reach it. Before the resume, the same subtest adds a status strand with `sleep 3600` and no sidecar, so the resume must replace it. After the resume it could read `eng.Status()` and assert that exactly one live strand matches `statusStrandDisplayName`, with a GUID other than the one added in the earlier subtest (`smoke_drivingsurface_test.go:138-158`).

Tool calls:
- A grep for `"loom", "resume"` across the `internal` tests, to confirm no other test runs the real resume verb's wake path and checks the strand. My context held only the files named in the diff.
