```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per-behaviour list**

1. `dobbyStartError` (started or live passes; refused, failed or unreported is an error naming the worktree and cause). Pinned by `TestDobbyStartError` in `internal/orchcli/startdobbies_test.go`, which has a row for each outcome.
2. `ensureParentDobby` calls `start(parent)` with the parent worktree, once when the dobby is down and never when it is live. Pinned by `TestStartFirstPass` in `internal/battencli/cli_test.go`. `wantPairStarts` and the per-parent loop cover prime and beam parents.
3. Owner and landing-hold refusals still run before the pair start. Pinned by the refusal rows of `TestStartFirstPass`, which assert `pairStarts` is empty.
4. A failed pair start gives a warning naming `lym beam start root`. Pinned by the `FailedPairStartWarnsAndKeepsThePass` row.
5. The anchor move starts dobbies after the lock release and `Close`. Pinned by the call-order list in `internal/fabriccli/anchor_test.go:508-511`.
6. A restart failure is reported as `DobbyStartError`. Pinned by `anchor_test.go:776` with a faked `EnsureDobbies`.
7. `BeamUpWorktree` (the guard, resolving a non-prime worktree, `BeamUp` with an empty `BeamUpOptions`, returning `dobbyStartError`). **None.**
   - In a test binary it returns `ErrBeamUpUnderTest` at once, so no test reaches the location resolution or the prime-versus-beam branch.
   - Nothing in the tests asserts `ErrBeamUpUnderTest` either.
8. The `beamStart` loop and `beamStart` itself in `internal/fabriccli/anchordeps.go`. **None.**
   - Not pinned: joining failures across worktrees instead of stopping at the first; the `root` label for the prime; `cmd.Dir` being the prime's anchor; the failure text carrying the child's output.
   - `anchor_test.go:214` fakes `EnsureDobbies` wholesale, and `beamStart` returns an error under `testing.Testing()` before any of that logic runs.
9. The default `pairStart` in `ensureDobby` (nil selects `orchcli.BeamUpWorktree(c.location, parent)`). **None.** Every test sets `c.pairStart`, so the real default, including which location it passes, is never exercised.
10. The pawl `StartDobby` wiring, which now calls `orchcli.BeamUpWorktree(hub, slug)` in place of the removed `startBeamDobbyAlone`. **None.**
    - `internal/loomcli/wiring_seams_test.go:106` only records the seam as filled.
    - `internal/pawlengine` tests use a fake.
    - The old refusal-to-error behaviour of `startBeamDobbyAlone` had no test either, and the new path would only return `ErrBeamUpUnderTest` in a test.

**Findings**

- **MEDIUM, design:** `internal/fabriccli/anchordeps.go:258-266` and `:287-314`. The restart loop and `beamStart` have no coverage: joining failures, the `root` label for the prime, `cmd.Dir`, and the child's output in the error text. The `testing.Testing()` guard returns before any of that logic runs. A test could build the label and args and join the failures in a pure helper, with the process run injected, or run the test binary as the fake `lym` through the repo's usual re-exec harness.
- **LOW, design:** `internal/orchcli/start.go:479-496`. `BeamUpWorktree`'s resolution of the prime versus a beam worktree and its `BeamUp` hand-off are unpinned. Factor the location choice into a small function the test can reach, or inject the bring-up as a seam.
- **LOW, design:** `internal/battencli/cli.go:587-590`. The nil default of `pairStart` (`orchcli.BeamUpWorktree(c.location, parent)`) is never exercised.
- **LOW, design:** `internal/loomcli/pawlwiring.go:113-119`. The pawl `StartDobby` wiring to `BeamUpWorktree` is unpinned.

**Tool calls**
1. `grep` for `EnsureDobbies`, `DobbyStartError`, `StartDobby` and `beamStart(` in the fabriccli, loomcli and pawlengine tests. My context did not show which tests exist for the anchor move or the pawl seam.
