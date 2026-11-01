---
round: 1
focus:
  - "Every remaining lone dobby spawn: grep non-test code for `dobbyengine.EnsureLive` and `dobbyengine.SpawnDetached` callers outside `orchcli`. The settled scope says each goes through the one pair start or stops naming `lym s`; settle whether any still starts a dobby without its orch."
  - "`orchcli.BeamUpWorktree` in `internal/orchcli/start.go`: does it match `lym beam start` for the prime and for a non-prime worktree (location resolution, runs strand, orch watcher binding)? Check `dobbyStartError` for outcomes beyond started and live, such as refused or an owner on another machine, and that `BeamUp` is safe to call when the dobby is already live."
  - "`testing.Testing()` guards in production code (`BeamUpWorktree`, `beamStart` in `internal/fabriccli/anchordeps.go`): settle whether the repository's rules allow a test-binary check in non-test code, and whether the two error paths leave the callers' tests covering real behavior."
  - "The anchor move restart in `internal/fabriccli/anchordeps.go`: the loop over worktrees joins failures instead of stopping at the first; check that the label mapping to `root` for the prime matches `lym beam start`, that `cmd.Dir` is right, that `cmd.Output()` leaves stderr unread so the error text holds the failure, and that the old `EnsureLive` refusal handling (owner on another machine) is not dropped."
  - "`ensureParentDobby` in `internal/battencli/cli.go`: the owner and landing-hold refusals must still run before the pair start, and the error wording must still name `lym beam start`. Check the `pairStart` seam's nil default and the updated tests in `cli_test.go`."
  - "`startBeamDobbyAlone` removal in `internal/loomcli/pawlwiring.go`: check no caller of its refusal behavior is lost, and that `BeamUpWorktree` returning `ErrBeamUpUnderTest` in tests does not hide a path the pawl tests used to exercise."
  - "The new `PATTERN-dobby-orch-pair` entry in `PATTERN.md`: check it against the repository's constraints document for required form (a `(test)` marker, a `pattern/` background file) and that its wording covers the child-process route the diff takes."
  - "Doc and comment changes in the touched `doc.go` files and godoc: compliance with the repository's comment conventions, and that the docs the repository names for this kind of change land in the same diff."
  - "Test economy in `startdobbies_test.go`, `cli_test.go` and `anchor_test.go`: new tests that duplicate existing coverage, and new behavior (`BeamUpWorktree`, `dobbyStartError`, the `beamStart` loop) with no coverage."
---

Seeded from the board entry in the decision record, which is the only answer key (no plan), and from the diff's file list.
