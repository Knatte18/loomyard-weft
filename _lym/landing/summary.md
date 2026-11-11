# Retire _launchers: fabric stops writing it, lym cleanup sweeps it

fabricengine wrote launcher scripts for every pair under `<hub>/_launchers/<anchor>/<slug>/`: `ide`, `run` and `fabric-checkout`, each a `.sh` or a `.cmd`.
It also wrote an `ide-menu` script per anchor, and kept those files in step through `add`, `remove`, `prune`, `reconcile` and `anchor --set`.
Nothing in lym runs, reads or documents those scripts any more, since terminals open through reed.
Leftovers of pairs that no longer exist pile up in every hub.
This change removes the writer and its teardown, and adds a one-time sweep of the directory to `lym cleanup`.

## What changes

- **fabricengine writes no launchers.**
  `launchers.go` and `launcher_content.go` are deleted.
  `Add`, `EnsureAnchor`, `AnchorChange.Apply`, `Reconcile`, `Remove`, `Prune` and the origin-ended teardown no longer touch `_launchers`.
  Gone with them:
  - `LauncherDir`, `PairParts.Launchers` and `RemoveStepLaunchers` (`"launchers"`);
  - `ReconcileActionLaunchersRestored` (`"launchers_restored"`);
  - the `removeMenuLauncher` step of an anchor move;
  - `pruneEmptyAncestors` and `refuseUncontainedPath`, whose only callers were the launcher paths.
  A pair whose only local remnant was its launcher directory no longer counts as a pair part.
  So `lym fabric remove` and `lym batten cancel` treat it as nothing left.
- **`lym cleanup` gains a `launchers` segment**, run after `temp`.
  It calls the new `fabricengine.SweepRetiredLaunchers` over the hub.
  The sweep removes the script names fabric wrote and any `.code-workspace` file.
  Then it removes each directory that empties, the `_launchers` container last.
  Anything else, a link included, is kept and listed under `kept`, and the directories above it stay.
  Every removal goes through the destructive gate under the container, non-recursively and without following a link.
  The dry run lists the same paths and removes nothing; a hub without the directory lists nothing.
- **Docs.**
  The fabricengine and cleanupcli package docs, `lym fabric` help, `docs/overview.md`, `docs/shared-libs/lymcwd.md`, `docs/sandbox-hub.md`,
  the fabric sandbox suite's F14 and F17 scenarios, `PATTERN-cwd-resolution` and `PATTERN-fabric-boundary` now describe the retired container.

## Design decisions

- **`_launchers` stays a reserved hub name.**
  The board entry asked to take the reserved name out of fabricengine.
  It stays in `HubReservedNames`, beside the retired `_portals`.
  A pair named `_launchers` would sit at the container's path, and the cleanup sweep would then delete its `run.sh` or `ide.sh`.
  Keeping the reservation costs nothing and rules that out while any hub may still hold the directory.
- **The sweep removes known files instead of the whole tree.**
  Removing `_launchers` recursively would also delete anything an operator had put there.
  The sweep follows the existing `_portals` sweep: it removes only what fabric wrote and names what it keeps.
- **The reconcile sweep of the retired workspace file under `_launchers` is unchanged.**
  `lym fabric reconcile` still removes `<hub>/_launchers/<anchor>/<prime>.code-workspace`, and the workspace regeneration still warns about it.
  The cleanup sweep removes the same file when reconcile has not run.
- **A pre-existing loomcli failure is left unfixed, per the parent's recorded decision.**
  `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` fails on this branch's base.
  The test expects the driver log to name the poisoned status file's invalid state.
  Since `d1535412` the log names the loop's `loop-exited` interrupt instead.
  It is out of this task's scope and filed as the board note `smoke-died-driver-wording`.

## Checked before the change

No desktop entry under `~/.local/share/applications`, `~/Desktop` or `~/.config/autostart` points at `_launchers` or one of its scripts on this machine.
Neither does any hub workspace file or any worktree `.vscode` folder of the five installed hubs.
Windows shortcuts were not checked.

## Open risks

- `hubmutation_integration_test.go`'s `Remove_RefusesDriftedLauncherDirLinkWithRefusalObject` was deleted with the launcher teardown it drove.
  It was the only test that asserted the `refusal` object, with `check`, `what`, `target` and `reason`, on a real `lym fabric remove` refusal envelope.
  A replacement would need another refusal trigger, such as a drifted scratch or junction link.
