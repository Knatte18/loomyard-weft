# Automatic config reconciles keep keys the template lacks

Fixes issue #2. A hub-wide reconcile run by an older lym binary removed keys that a newer binary had written to the committed default layer.

## Problem

The binary-change walk (`hubreconcile.Ensure`), `lym fabric clone` and `lym fabric reconcile` called `configsync`. That pruned every key the running template lacked from the board's base files and from each worktree file, and committed the result.
A binary built before a key existed could not tell that key from a stale one.
On a machine running such a binary, the first start verb removed `beam.pull_request` and `landing.pr_reviewer` from the board.
Runs then went through PR-Review against the operator's setting.

## Change

- `configsync.Options` and `configsync.HubWideOptions` gain `PruneUnknown`.
  Without it, a present base or worktree file loses only the keys that a retired key of its module covers.
  Any other key the template lacks stays, and `Result.Preserved` reports it on dry runs and applies alike.
  A new helper, `pruneSparse`, does both: it drops retired keys with the newly exported `configengine.DropRetired` and runs `yamlengine.Prune` to find the keys it keeps.
- Only `lym config reconcile` sets `PruneUnknown`, for both the worktree form and `--layer default`, so removing an unknown key stays an explicit operator step, as the issue asks.
  The walk, clone and `fabric reconcile` keep the zero value.
  The verb's help now says it is the one way to remove such a key.
- A local override file is still pruned of every unknown key on any reconcile, because its load refuses an unknown key. Keeping one would only break the load.
- The walk logs `preserved` beside the other per-module fields.
  A kept key still produces the existing Warn on load, which names `lym config reconcile --layer default --apply` as the way forward.
- Docs: `hubreconcile`'s package doc gains a "Keys the template lacks" section, and the `configsync` header and function docs now describe the option.
  `PATTERN-config-layers` (index line and background) records the new rule: only `lym config reconcile` removes a base or worktree key that the template lacks and no retired key covers.

## Design decisions

- Retired keys are still pruned automatically. The running binary declares them, so it knows their values are dead; every load already drops them.
  Keeping them would leave stale entries on every board for no gain.
- The fix applies to every automatic reconcile, not only when the running binary is older than the stamp's build.
  A build has no order that tells "older" from "newer", and the explicit verb already covers removal.
- The worktree layer gets the same protection as the default layer. Worktree files are committed and travel between machines too, so an older binary walking a pair could strip them in the same way.

## Tests

- `configsync`: a new test, `TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked`, pins that a board file keeps and reports an unknown key, drops a retired key beside it, and loses both under `PruneUnknown`. It also pins that a dry run writes nothing.
  `TestReconcileAll_SparseLayers` gains rows for a worktree file that keeps an unknown key, or drops a retired one, without `PruneUnknown`. The existing unknown-key rows now set the option, and the local-file row pins that local files are pruned regardless.
  Rows in `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` that relied on an unknown key being pruned now use loom's retired `pr_review`.
- `hubreconcile`: the stale-build walk seeds a fabric key that the template lacks and asserts it survives, both on a marked board and through the one-time base strip.
  Fixtures that needed some hub-wide rewrite to force a board commit now seed landing's retired `publish_verify` (or darn's retired `verify`) instead of an unknown key.
- `fabriccli`: the reconcile board-failure test seeds landing's retired key for the same reason.
