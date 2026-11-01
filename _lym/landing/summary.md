# Start a worktree's dobby only with its orch

A dobby that comes up without its orch posts notices nobody reads.
`lym batten run`, run by hand, did exactly that: after its pass, `ensureParentDobby` in `internal/battencli/cli.go` spawned the parent worktree's dobby through `dobbyengine.SpawnDetached` when it was down, and nothing started the orch, so only a later `lym beam start` brought the pair back together.

Every path that starts a dobby now brings the worktree up the way `lym beam start` does, through `orchcli.BeamUp`.

## What changed

- `orchcli.BeamUpWorktree(prime, worktree)` is new.
  It resolves the named worktree from the prime, the prime's own name naming the prime, runs `BeamUp` without `Prepare`, and returns a dobby start that was refused or failed as an error carrying its detail, since `BeamUp` itself only warns there.
  In a test binary it returns `ErrBeamUpUnderTest` and starts nothing, as `dobbyengine.SpawnDetached` does.
- `ensureParentDobby` (batten's first pass and every hand pass) keeps its owner, landing-hold and liveness checks, and brings a down parent up through `BeamUpWorktree` in place of the lone spawn.
  Its seam `dobbySpawn` became `pairStart`, which takes the parent worktree's name.
  A live dobby is still left alone, so an orch that died beside a live dobby is not revived here.
- Pawl's `StartDobby`, the Beam-Await restart after a refused land request, calls `BeamUpWorktree` for the beam; `startBeamDobbyAlone` is removed.
- The anchor move's `EnsureDobbies` in `internal/fabriccli/anchordeps.go` runs `lym beam start <worktree>` (`root` for the prime) as a child process per worktree it restarts, joining every failure into `dobby_start_error`.
  It runs once the parent lock is released and the anchor change closed, since `lym beam start` runs the hub's origin reconcile.
- `PATTERN.md` gains `PATTERN-dobby-orch-pair`.

## Design decisions

- `BeamUp` is run without `Prepare`: the hub-wide origin reconcile, prime claim and owner pauses belong to `lym beam start` itself, and pawl's Beam-Up step already runs `BeamUp` this way.
- A refused start of a beam during an anchor move, such as a beam another machine owns, now shows up in `dobby_start_error`; `dobbyengine.EnsureLive` used to drop such a refusal without a word.
  The verb still exits 0.
- `fabriccli` runs `lym beam start` as a child rather than calling `orchcli.BeamUpWorktree`: `hubforge`, the test fixture that the orch and creel packages' tests import, imports `fabriccli`, so that import would make an import cycle in those tests.
  The child is the full `lym beam start`, origin reconcile and prime claim included.
- Whether a running dobby revives a missing orch is left to the `dormant-orch` note, as the board entry says.

## Tests

`TestChangeAnchor` now expects the dobby restart after the parent lock is released and the change is closed.
`TestStartFirstPass` now asserts that the pair start receives the parent worktree's name, and that a refused slug starts no pass and brings no worktree up.
The new `TestDobbyStartError` pins the outcome translation: a started or live dobby passes, while a refused, failed or unreported start becomes an error that names the worktree and the cause.
