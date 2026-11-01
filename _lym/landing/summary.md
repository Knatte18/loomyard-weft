# Start a worktree's dobby only together with its orch

A hand `lym batten run <slug>` after the hub's tmux server was killed brought the parent worktree's dobby up alone.
`ensureParentDobby` spawned it, nothing started its orch, and the dobby's mail to the orch had no reader.
Now every path that brings a worktree's dobby up brings its orch up with it, the way `lym beam start` does.

## What changes

- `orchcli.BeamUpWorktree(prime, worktree)` is new: it runs `BeamUp` without `Prepare` for the named worktree (the prime's own name names the prime).
  It returns `BeamUp`'s refusal or failure, and a dobby that was refused or failed to start, as an error carrying its detail.
  It returns `ErrBeamUpUnderTest` in a test binary, so no test starts a real dobby, tmux session or agent.
- `battencli`: `ensureParentDobby` takes a `start(parent)` seam in place of the dobby spawn, wired to `orchcli.BeamUpWorktree`.
  Its owner and landing-hold refusals and the `lym beam start` way forward are unchanged.
- `loomcli`: a beam's dobby restart after a failed Beam-Await re-check goes through `orchcli.BeamUpWorktree`.
  `startBeamDobbyAlone` is deleted.
- `fabriccli` (`lym fabric anchor`): the dobbies a move ends start again through one `lym beam start <worktree>` child per worktree (`root` for the prime), waited for one by one.
  A failed child does not stop the others; `dobby_start_error` joins every failure with the child's output.
- `PATTERN.md` gains `PATTERN-dobby-orch-pair`: never `dobbyengine.EnsureLive` or `SpawnDetached` alone to start a worktree's dobby.
  The affected package docs are updated to match.

## Design decisions

- The anchor move starts its dobbies through a child process because `fabriccli` cannot import `orchcli`: a test fixture imports `fabriccli` from the orch packages' own tests, so the import would cycle.
  The child runs from the prime's anchor, and fails without starting anything in a test binary.
- The move now releases the parent lock and closes the change before it restarts the dobbies.
  A `lym beam start` child runs the same refusals as a hand start and would otherwise meet the move's own standing anchor change.
- `BeamUp` itself still starts the dobby through `dobbyengine.EnsureLive`, as the first step of the pair it brings up.
- Out of scope, as the task says: whether a running dobby brings back an orch that has gone is note `dormant-orch`.

## To check by hand

- The anchor move's restart through `beamStartChild` (`internal/fabriccli/anchordeps.go`) is not exercised against a real `lym beam start`: the child is refused under test, and the tests cover only the loop in `startDobbiesThroughBeamStart` and the order of the unlock and the restart in `moveAnchor`.
  Run `lym fabric anchor` on a hub with live dobbies and confirm each comes back with its orch (`lym a` finds the orch in reed).
- Kill the hub's tmux server, run `lym batten run <slug>` by hand, and confirm the parent worktree's dobby and orch both come up.
- The same for a beam whose Beam-Await re-check fails: the restarted dobby has its orch.
