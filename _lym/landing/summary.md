# Sweep read-only leftovers a killed test leaves under the temp root

A test that made a directory read-only and restored it only in `t.Cleanup` left the directory read-only when its binary was killed.
`internal/lymtemp`'s dead-owner sweep then failed to remove the run directory holding it,
so the `temp` segment of `lym cleanup --apply` failed on machine state.
That failure is what broke `TestCleanupVerb_ScratchSegmentSweepsTheAnchorOfASubpathHub` in an unrelated run's verify gate.

## What changes

- **The sweep makes a dead tree owner-writable first.**
  `Root.Sweep` now removes a dead run directory through `removeTreeMakingWritable`.
  It walks the tree and adds owner read, write and search permission to every directory, setting the bits before it lists the directory,
  then calls `os.RemoveAll`.
  Files and links keep their bits, and a link is never followed.
  An entry that vanishes mid-walk under a concurrent sweeper is skipped, and the retry on the not-empty symptom is unchanged.
  A new `TestSweep` row pins a dead run directory holding a read-only directory and an unreadable one below it.
- **One kit for the chmod-and-restore pattern.**
  The new `internal/testkit/permkit` exports `Chmod(tb, path, mode)`.
  It sets the bits and registers a cleanup that restores the bits the path had before.
  Every test that chmodded a path and restored it in `t.Cleanup` now calls it,
  across webster, fabric, config, shuttle, machine, vscode, friction, batten, burler, reed, logger, shed, impactset, gate and `tools/forkaudit`.
  The local `makeReadOnly` and `makeUnreadable` helpers in fabricengine, impactset and forkaudit are gone.
  Tests that chmod a file to `0` with no restore are left alone, since a file's own bits never block its removal.
- `PATTERN-test-isolation` and its `PATTERN.md` line name the kit and the sweep's owner-writable pass.

## Design decisions

- The board entry calls the fix "the scratch sweep of the cleanup verb", and its title says "the scratch root".
  The leftovers sat under `/tmp/lym-1000`, the lym temp root, and the cleanup verb's `temp` segment is the one that failed.
  So the fix is in `lymtemp`'s sweep, which every process start also runs, not in the `.scratch` segment of `cleanupcli`.
- The restore puts back the path's original bits rather than a fixed `0o755`.
  It drops its own error, so a test whose code under test removed the path still passes.
- `permkit` imports the standard library only, so `lymtemp`'s own tests use it without an import cycle through `tmuxkit`.
- No scan forbids a raw chmod-and-restore outside the kit; review and the `PATTERN-test-isolation` rule hold it.
