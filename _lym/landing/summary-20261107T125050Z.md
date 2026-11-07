# fabric clone: render the prime later and print a failed render

`lym fabric clone` told heddle's worktree seam that the prime was made right after wiring its junctions.
That was before `configsync.ReconcileAll` wrote the prime's per-worktree module configs and before their commit.
It also dropped the warning text `fabricengine.NotifyWorktreeMade` returns, so a failed render showed only in the log at Warn.

## What changes

- `fabriccli.CloneAndWire` now fires the made event once the per-worktree module configs are committed.
  The prime is then set up as an added pair is when its seam fires.
  The event still fires before the `hubreconcile.Ensure` walk, so a walk failure that fails the clone still leaves the prime rendered.
- The event's warning is kept in the new `fabricengine.CloneResult.SeamWarning`.
  `lym fabric clone` prints it as the envelope's `seam_warning` field, which is always present and empty when the render succeeded.
  The warning names the prime and the way forward, `run lym heddle render in that worktree`.
  The readable terminal summary shows it, since it prints every key ending in `warning`.
- `hubforge.RegisterTeardown` is exported, so a test that calls `CloneAndWire` by hand tears down the junctions it wired.

## Design decisions

- The event fires before the walk rather than last.
  The walk's board push is handed to a detached sync, and a render that waits on the Bolt checkout gains nothing by running beside it.
  Firing before the walk also keeps a walk failure from leaving the prime unrendered.
- The clone carries the seam warning in its own field, as `fabric pull` does with `seam_warning`, rather than folding it into `warning`, which holds the shortname warning.

## Open risks

- The cause the operator hit is not confirmed: no clone log holding a seam warning was found on this machine.
  Both differences the entry names are fixed.
  A render that ends with warnings, such as a target missing from the warp exclude list, still returns no error from `heddleengine.Seam.Made`.
  Those warnings reach only the log, so a clone that hits one still prints no `seam_warning`.

## Tests

A new subtest of `TestWorktreeSeam_Events` in `internal/fabricengine` clones with a failing seam and asserts that the clone succeeds and that `CloneResult.SeamWarning` names the prime and `lym heddle render`.
