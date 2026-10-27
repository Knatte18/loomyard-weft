# A read creates no directory: hub readers stop breaking a half-built pair

A hub-wide reader polling a pair that `Topology.Add` was still building could leave a real `<warp>/_lym` behind, and the junction check then refused the pair with "warp repo already contains a real _lym".
The `lym batten overview` refresh (`orchcli.HubOrchs`) and the tender's `hubOrchs` hit this: `fabricengine.ReadOrigin` read through `state.ReadJSON`, whose `os.MkdirAll` created `<warp>/_lym/fabric`.
A read during staging also left a stray `.weft` directory under `.fabric-wt-staging-*`.

## What changes

- `state.ReadJSON` no longer creates a directory. A file whose directory is absent reads as a missing file.
- `state.ReadJSONNoMkdir` is removed; its callers move to `ReadJSON`. It carried the same error shape, so `ErrRead` and `ErrDecode` now come from the one reader.
- Callers that created a lock directory only so a read could lock it no longer do: `battencli`, `loomengine` (`CheckSeed`, `VerifySeedOwnership`), `shedverbs status` and `websterengine.LoadState`.
- `fabricengine.ReadOrigin` builds its lock path without creating the `.weft` directory or seeding its exclude. `WriteOrigin` keeps the creating path.
- `orchcli.HubOrchs` skips a worktree under a `.fabric-wt-staging-*` directory, and a pair whose `_lym` is not yet a link, before reading its origin. The skip is silent and logs no warning.
- `Topology.Add` wires the junctions right after the weft worktree exists, ahead of the local-branch marks and the launchers, which shortens the window in which a warp worktree has no junction.
- `internal/state/doc.go`, `internal/orchcli/doc.go` and `contracts/specs/loom-status-spec.md` follow the new reader.

## Design decisions

- **A read with no lock directory reads without the lock.** `ReadJSON` still takes the shared lock, but skips it when opening the lock file fails with not-exist; any other lock error still fails the read. No writer can hold a lock in a directory that does not exist, since writers create it before locking, and every write replaces the file by atomic rename, so an unlocked read sees a whole file.
- **The staging and junction checks live in `fabricengine`** as `InWorktreeStaging` and `LymJunctionWiredHere`, so `orchcli` names no fabric path and runs no git.
- **Lock files a read creates inside an existing directory are unchanged.** The read lock has always done that, and only directory creation is in scope.
- **Hold test lives in `fabricengine`.** Holding `Add` between weft creation and wiring from `orchcli` would need an exported test seam in production code, so `add_hubreader_integration_test.go` holds `Add` through an unexported `beforeWire` hook and runs the path `HubOrchs` takes (`List`, `lymcwd.ResolveWorktree`, `ReadOrigin`) over every listed worktree. A separate `orchcli` case covers `HubOrchs` skipping an unwired pair.

## Check by hand

- A caller that wrote into a directory with a raw `os` call, relying on an earlier read to have created it, would now fail. A sweep found none: writers go through `WriteJSON`, `UpdateJSON` or `fsx.AtomicWriteBytes`, which create their directory.
- `TestEnforcement_MarkdownLinks` fails on `main` from the bench data commits under `docs/research`, not from this change. Task `docs-link-skip-bench-data` fixes it on `main`.
