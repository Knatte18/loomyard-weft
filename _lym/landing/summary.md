# Prime dobby starts coverage-map upkeep on remaining work

The prime dobby now starts a coverage-map upkeep pass whenever work remains, and an anchor move of the prime ends a running pass instead of waiting for it.
This replaces the landing-count trigger (`test_map_every`, `test_map_batch`), so a backlog drains on its own.

## What changes

- `dobbyengine` starts a pass as `lym gate testmap --upkeep` when no pass is live, no anchor move of the prime stands, and one of these holds:
  there is no exit record, the prime's HEAD differs from the planned HEAD in the last exit record, or that record names queued work.
- On a hub with fewer than two slots, queued work alone starts no pass, since such a pass takes no slot.
- The dobby starts no pass while a pass it started is still its live child, registered or not.
- A `--upkeep` call whose own registration as `runproc.RoleTestMap` fails exits before it plans.
  The dobby logs that exit once per episode and starts a pass again on a later tick.
- `test_map_every`, `test_map_batch` and the dobby state's landing count are retired.
  The landing count stays readable in an old state record.
- New dobby key `test_map_upkeep`, `configengine.CategoryNone`, true by default.
  False switches passes off.
- `fabriccli`'s `dobbyPassInFlight` no longer counts a `runproc.RoleTestMap` process, so `lym fabric anchor` of the prime does not wait for a pass.
  The existing end of the prime's hub-owner processes (`EndDobbies` through `runproc.EndSparing`) ends the pass.
- `orchcli`, `runproc` and the docs (`docs/overview.md`, package `doc.go` files, `pattern/PATTERN-test-speed.md`) follow the new wording.

## Design decisions

- `test_map_every: 0` was the off switch, so it migrates to `test_map_upkeep: false`.
  Any other `test_map_every` value is dropped, as is `test_map_batch`.
- The migration is `configreg.Module.Migrate` on beam, not a `configreg.Reinterpretation`.
  The reconcile walk's `pruneSparse` drops the units a retired key covers before reinterpretations run, so a reinterpretation never sees the key.
  beam is per-worktree, so the same migration also covers the prime's `_lym/config/beam.yaml` and beam's local override when `configsync.ReconcileAll` reconciles them.
- `configengine.Spec` gains a per-layer migrate hook.
  `configengine.Load` applies it to each layer in memory before merging and before `DropRetired`.
  A prime dobby re-exec'd onto the new binary therefore reads `test_map_every: 0` as off before any walk has run.
  `dobbyengine` owns the rewrite: its `ConfigSpec` sets the hook, and beam's `Migrate` calls the same function.
  The in-memory pass logs nothing; the walk logs the rewrite when it writes it.
- A pass plans once at HEAD and ends at its next job or wake once HEAD moves; the next start plans again.
- Queued work leaves out a package skipped for a dirty file, a package whose recorded failure names the current HEAD, and every spawn-marked test.
  A dirty prime or a broken commit therefore does not restart passes in a loop.
- Only a `--upkeep` call reads and writes the exit record.
  A hand caller passing the flag runs as a pass and blocks the dobby's start while it lives.
- A call naming no package and lacking `--upkeep` is not registered, so it can run beside a pass.
  It wastes work, and the build lock keeps the two off the same package.

## To check by hand

- Not run here: a live prime dobby over a hub whose `beam.yaml` holds `test_map_every: 0`, to see that no pass starts before any walk has run.
- Not run here: an anchor move of the prime while a real pass is running, to see that it ends the pass and does not wait for the queue.
