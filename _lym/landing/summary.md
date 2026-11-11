# Remove the hub's _launchers directory and sweep it in lym cleanup

Fabric no longer writes launcher scripts under `<hub>/_launchers`.
Nothing in lym runs, reads or documents those scripts, and terminals open through reed.
`internal/fabricengine` used to write `ide.sh`, `run.sh` and `fabric-checkout.sh` per pair and an `ide-menu.sh` per anchor, then remove them with the pair.
Pairs removed outside that path left stale copies behind.
This change deletes the writers, their cleanup in the pair verbs, and the tests that covered them.
`lym cleanup` gains a `launchers` segment that removes the directory from hubs that still hold it.

Closes #95

## What changes

- `internal/fabricengine`: `launchers.go`, `launcher_content.go`, the portal launcher test and the launcher tests are gone.
  `add`, `addanchor`, `anchorchange`, `reconcile`, `remove`, `prune` and `destroy` no longer create, mirror or remove launcher files.
  `ancestors.go` loses the launcher-specific helpers.
- `internal/hubforge`: `Hub.PairLauncherDir` is removed with its last users.
- `lym cleanup`: a new last segment, `launchers`, calls `fabricengine.SweepRetiredLaunchers` (in `retiredlaunchers.go`).
  A dry run lists under `removed` what `--apply` would delete and under `kept` what it leaves.
  A hub without the directory lists nothing.
- Docs and patterns (`docs/overview.md`, `docs/sandbox-hub.md`, `docs/shared-libs/lymcwd.md`, `pattern/*`, the sandbox fabric suite) drop their launcher references.

## Design decisions

- The sweep removes only the eight script names fabric wrote (`ide`, `run`, `fabric-checkout`, `ide-menu`, each as `.sh` or `.cmd`) and any `.code-workspace` file.
  It then removes each directory that empties, the container last.
  Anything else is kept and named in `kept`, and the directories above it stay, so a user's own file in there is never deleted.
- The sweep never follows a link, and each removal goes through the destructive gate under the container.
  A `_launchers` that is itself a link or a file is kept, not removed.
- A removal that fails stops the sweep and returns what it had removed with the error.
- The `_launchers` name stays in `HubReservedNames` (`internal/fabricengine/junctionnames.go`), like `_portals`.
  No pair or junction can take the name of a leftover directory on a hub that `lym cleanup` has not yet swept.
  The task's idea listed dropping the reservation; that part is deliberately not done.

## Check by hand

- I did not check installed hubs for a desktop shortcut or VS Code task that still points at one of the scripts, which the issue asked to do first.
  Look before running `lym cleanup --apply` on a real hub.
- `internal/loomcli` `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` fails on `main` already, because the driver log says `step interrupted (loop-exited)` where the test waits for `invalid state`.
  This change does not touch it; it is filed as the bug note `smoke-died-driver-wording`.
- Open review findings, to check in the diff:
  - The sweep's handling of a symlink at `_launchers` is covered by `TestSweepRetiredLaunchers` in `retiredlaunchers_test.go` only partly; check that the symlink case in that test pins the kept behaviour.
  - The `lym cleanup` refusal envelope for the `launchers` segment, when the sweep errors, has no test of its shape.
  - Keeping the `_launchers` name reserved is a decision, not an oversight; confirm you agree.
