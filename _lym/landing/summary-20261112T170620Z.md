# Dobby starts coverage-map upkeep on remaining work; anchor moves end it

The prime dobby used to start `lym gate testmap --max <test_map_batch>` every `test_map_every` landings.
A backlog therefore drained only when landings came.
It now starts an upkeep pass, `lym gate testmap --upkeep`, whenever the coverage maps have work left, and an anchor move of the prime ends a running pass instead of waiting for it.

## The start rule (`internal/dobbyengine`)

`testMapStep` starts a pass while `test_map_upkeep` is on and no pass is live, when any of these holds:

- no exit record exists;
- the prime's HEAD differs from the last exit record's planned HEAD;
- that record names queued work, and the hub has at least two gate slots.

A pass on a hub with fewer than two slots takes no slot, so queued work alone starts nothing there.
That keeps a one-slot hub from restarting passes at the same HEAD.

A pass counts as live while a `runproc.RoleTestMap` process is registered for the hub, a hand-started `--upkeep` call included.
It also counts as live while the pass this dobby started is still its child, registered or not.
That closes the window between the spawn and the pass's own registration.
The tick that reads the child's exit starts nothing, so a pass whose registration failed is started again on a later tick.
A failed exit is logged once per episode, in `dobby.log` and at Warn, as a failed tick is, and the first clean exit ends the episode.
The step already ran only while the tick schedules, so no pass starts while an anchor move stands.

New seams carry the inputs: `StartTestMap` now returns the child's exit channel, and `TestMapExitRecord`, `PrimeHead` and `GateSlots` are added.
Only the prime's wiring sets them.
The state record's landing count, `landings_since_test_map`, is gone from `State`.
An old record holding it still reads, and `UpdateState` keeps the key.

## The config key and its migration

`test_map_every` and `test_map_batch` are now retired keys of beam.yaml, and `test_map_upkeep` (`CategoryNone`, true by default) replaces them.
`dobbyengine.MigrateConfig` rewrites `test_map_every: 0`, the old off switch, to `test_map_upkeep: false` in its place, and drops any other value.
A document that already sets `test_map_upkeep` keeps that value.
It runs in two places:

- beam's `configreg.Module.Migrate`, so the reconcile walk writes and logs the rewrite.
  This is a migration rather than a `configreg.Reinterpretation`, because the walk prunes retired keys before reinterpretations run.
- a new `configengine.Spec.MigrateLayer` hook that `dobbyengine.ConfigSpec` sets.
  `StackLayers`, and so `Load`, runs the hook over each present layer in memory before the merge and before `DropRetired`, discarding its lines.
  So a prime dobby re-exec'd onto this binary over an unreconciled `beam.yaml` or local file that holds `test_map_every: 0` reads upkeep as off.

## The anchor move (`internal/fabriccli`)

`dobbyPassInFlight` now spares a registered `runproc.RoleTestMap` process as it spares push and sync, so the move no longer waits for a running pass.
The move's existing `EndDobbies` (through `runproc.EndSparing`) ends the prime's hub-owner processes, the pass among them.
The pass has no batch cap any more, so waiting for it could take a long time.

## The process start (`internal/beamcli`)

`startTestMap` starts `lym gate testmap --upkeep` and no longer registers the child, since the verb registers itself.
It keeps the tail of the child's output and delivers the exit code with the error text that output carries.

## Tests

- `dobbyengine`: `TestTestMapStep` covers each start-rule branch, the hold on a live child and on a registered pass, the once-per-episode log, and each failed read.
  `TestMigrateConfig` and new `TestLoadConfig` rows cover unreconciled worktree and local layers.
  The anchor-move and adopt-hold loop tests use the new seams.
- `configengine`: `TestLoad_MigrateLayer` covers the hook on each layer, files left unchanged, and an `ErrInvalid` failure that names the file.
- `configreg`: a `TestPrepareRunSettings` row pins beam's Migrate.
- `beamcli`: `TestStartTestMap` covers the command line, the exit not being delivered while the child runs, and a clean and a failed exit.
  The wiring test checks that only the prime sets the upkeep seams.

## Design decisions and bounds

- A `fabriccli` test of the spared role would need a real-process harness that the package lacks, so none was added.
  The change is one more role in the existing spare list.
- Bound: if the dobby re-execs after a spawn but before the pass registers, the new image no longer holds the child.
  A second start in that moment is refused by the pass's own registration check.
- `orchcli`'s prose still calls the `RoleTestMap` process "the test-map refresh".
  Its behavior, ending that record on the prime's stop, is unchanged.
