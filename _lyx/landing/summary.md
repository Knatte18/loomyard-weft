# One hub workspace file listing prime, Bolt and every pair

The operator opens a hub through one VS Code workspace file, `<hub>/<RepoName>.code-workspace`, replacing `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`.
It lists the prime's anchor directory first, named after its checked-out branch (the directory name on a detached HEAD), then `_board` as `Bolt`, then every live task pair at its anchor directory, named by slug and sorted in byte order.
A finished pair drops out again.
The `_portals` container, whose only consumer was the old workspace, is retired.

## What changes

- `fabricengine` gains one regeneration function that resolves the prime, lists the folders, builds the file through `vscode.BuildWorkspace` and writes it only when its bytes changed, recording `KindFileWritten`.
  `ideengine`'s prime spawn calls the same function, so the spawn and the verbs cannot write different files.
  An unresolvable prime still degrades the spawn to the bare folder; unreadable settings, a lock timeout or a failed write are returned as the spawn's error.
- `Clone`, `Add`, `AddLocal`, `Remove`, `Prune` with apply, `Checkout` and `Reconcile` regenerate the file, on every exit past their pre-flight checks, rollbacks and partial failures included.
  A regeneration failure never fails or rolls back the verb: it logs a Warn naming `lyx fabric reconcile`.
- Listing is by worktree, not by umbrella: local pairs (`fabric add --local`) and pairs forked from pairs are listed, and an entry the listing cannot resolve is skipped at Debug so one half-created pair does not drop the others.
- `_portals` is no longer created, restored or removed by any verb; the portal helpers, the `hubforge` accessor and their tests are gone.
  The name stays in the hub-reserved set.
- `lyx fabric reconcile` sweeps the retired artifacts through `destroy.go`: fabric-shaped portal links (judged by target path, including dangling ones) and the retired workspace file, which is removed as a link when one is planted there.
  Foreign entries under `_portals` and a directory at the retired workspace path are kept and reported in a new `hub_details` field of the reconcile envelope.
  Until the sweep has run, every regeneration logs a Warn naming `lyx fabric reconcile`.
- Reconcile's `portal_restored` action becomes `launchers_restored`, reported for an otherwise healthy pair whose launcher directory it restored.
  The launcher restore now skips local-only pairs, since `AddLocal` creates no launcher directory.
- `PATTERN-hub-containment`, `PATTERN-fabric-write-containment`, `PATTERN-hub-suffix`, the `fabricengine` doc, `lyx fabric reconcile` help and the sandbox fabric suite are updated to the new file and to `_portals` as retired.

## Design decisions to review

- The listing and the write run under one leaf lock, `<hub>/_board/.lyx/hub-workspace.lock`, with a 5 second wait; a timeout is a regeneration failure.
  A hub with no `_board` takes no lock, creates no directory and lists no Bolt, so parallel verbs there can leave a stale list until the next regeneration.
- The write is in place through the hub `os.Root`, with no temp file; a failure before the write writes nothing, and a truncated file is rewritten by the next regeneration because its bytes differ.
- The file is owned by lyx: every regeneration overwrites edits made in VS Code or by hand; workspace settings belong in the prime's `.vscode/settings.json`, which is still spliced in verbatim.
- Workspace files for other umbrellas and batch worktrees are out of scope; the function is written to be reused for them.
- `fabricengine` now imports `internal/vscode`; no cycle forms.

## To check by hand

- Open a real hub's `<RepoName>.code-workspace` in VS Code, then run `fabric add`, `fabric remove` and `fabric checkout` and watch the folder list update in the open window.
- A `Checkout` in the prime renames the first folder through the verb, which can restart VS Code extensions in an open window; confirm this is acceptable.
- Run `lyx fabric reconcile` on a hub that still holds `_portals` links and the old workspace file, and confirm both are gone, with a foreign entry kept and reported in `hub_details`.
- An open window with a pair folder whose directory vanished before the rewrite shows a missing folder until the next regeneration.
- A folder or setting added through VS Code's workspace UI is lost at the next regenerating verb.
