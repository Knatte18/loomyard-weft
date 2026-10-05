# Read board.yaml once per hub, from the board dir

The board is one per hub, but `board.yaml` (types, labels, README settings) was read from each worktree's own `_lyx/config/`.
A pair created before a vocabulary change kept its old copy, so its board writes re-rendered README with the old types.
Seen when the landing of `lyx-skills-and-stencils` rendered every `feature` note under "Other".

`board.yaml` now lives beside `fabric.yaml` at `<BoardDir>/_lyx/config/`, and every reader loads it from there, whatever worktree it runs in.

## What changes

- `configreg.Module` gains `HubWide`, set for `fabric` and `board`; every other module stays per-worktree.
  `configsync` and `configcli` branch on the field, never on a module name.
- `lyx board`, loom's landing board access and the `lyx ide` menu load the hub file.
  `boardengine.LoadConfig` keeps its told-`baseDir` signature; the CLI callers pass `fabricengine.BoardDir(hub)`.
  Loading stays strict, and an absent hub file is refused with a `lyx fabric reconcile` way forward.
- `lyx fabric clone` and `lyx fabric reconcile` reconcile every hub-wide module at the board dir (`configsync.ReconcileHubWideAt`, replacing the fabric-only reconcile).
  An absent hub `board.yaml` is seeded from the prime worktree's copy, else from the template.
  The template seed is silent in `clone` and a warning in `reconcile`, which says custom types and labels are not carried.
  The result names each file's seed source, and a present hub file is never replaced.
  Written files are committed and pushed in `_board`.
- `lyx config <module>`, `--set`, `--print` and `menu` address a hub-wide module at the board dir.
  The edit is made on a staging copy and validated through the strict load before it is written, so an invalid edit is refused and leaves the hub file untouched (`configengine.EditPath`, `StagingFile`).
  The write and commit run under the board write lock through the new `fabricengine.Bolt.CommitWritten`.
  If the commit or push fails, the file stays written and the error names it as uncommitted; the next board sync commits it.
  This also fixes `lyx config fabric`, which edited a per-worktree `fabric.yaml` nothing read.
- `lyx config reconcile` no longer reconciles per-worktree copies of hub-wide modules.
  With the hub file present and holding everything the copy holds, `--apply` deletes the copy and reports it as `retired`.
  A copy that holds keys or values the hub file lacks is reported as `divergent` and left alone, as is any copy while the hub file is absent.
- `PATTERN-hub-wide-config` states the rule.
  Package docs, `docs/overview.md`, `docs/shared-libs/configengine.md` and the board skill no longer place `board.yaml` per worktree.

## Design decisions

- The prime's copy seeds the hub file, not a pair's: the prime holds the live vocabulary, and a pair's copy is the stale one this change removes.
  The prime is resolved by the CLI caller from the worktree listing and told to `configsync`.
- Any worktree can now change the hub's board vocabulary through `lyx config board`.
  That is the same trust boundary as `lyx board upsert`, which already writes the hub board from any pair.
- Out of scope: the other config modules (they tune the run in the worktree that reads them), the general overlay (`config-overlay`), carrying per-worktree config changes to the parent (`config-reaches-parent`), and re-rendering README on a `board.yaml` edit.
- `configcli` is now a strict `configengine.Load` caller, so `cmd/lyx/configstrictness_test.go` and `pattern/PATTERN-config-strictness.md` change to match.
- Fixtures that loaded board config from the worktree anchor (`seedBoardTask` in `internal/battencli`) now load from the hub board dir.

## To check and run

- Deploy order: the new binary refuses `lyx board` on a hub until the hub `board.yaml` exists.
  After the deploy, run one `lyx fabric reconcile` on each hub.
- A hub whose default branch is checked out nowhere has no prime copy, so it gets a template-seeded `board.yaml` and loses custom types and labels until the hub file is edited.
  The `reconcile` warning is the only signal.
- Pairs running an older binary keep reading their own copy until torn down.
- Not run by hand: `lyx fabric reconcile` and `lyx config board --set` against a real production hub, including a failed push.
