# Decision record — board-config-per-worktree

## Goal

`board.yaml` describes the one board per hub, so every `lyx board` verb and every other board reader loads it from the hub's `_board/_lyx/config/board.yaml`, beside `fabric.yaml`, whatever worktree it runs in.
A pair created before a vocabulary change can then no longer re-render README or validate labels against a stale copy.

## Scope

In:

- Declaring which config modules are hub-wide, once, in the config registry: `fabric` and `board`.
- Every reader of board config (`lyx board`, loom's landing board access, `lyx ide` menu) loads it from the board dir.
- `lyx config` (edit, `--set`, `--print`, `menu`) addressing a hub-wide module reads and writes the board-dir file, and commits and pushes it in `_board`.
- Reconcile of hub-wide modules at the board dir (today `fabric` alone), generalized to `board`, including a one-shot seed of the hub file from the prime's existing per-worktree copy.
- `lyx config reconcile` removing a stale per-worktree copy of a hub-wide module once the hub copy exists.
- Docs and a new PATTERN entry for the hub-wide config rule.

Out:

- The other config modules: they stay per-worktree (survey in Decisions D1).
- The general two-layer overlay (`config-overlay`) and carrying per-worktree config changes to the parent (`config-reaches-parent`); both stay notes.
- Re-rendering README when `board.yaml` is edited; the next board write renders with the hub's config.

## Decisions

### D1 — Hub-wide modules are `fabric` and `board`, and only those

Decision: `board` joins `fabric` as hub-wide; every other module stays per-worktree.
Rationale: `board.yaml` describes the single `_board`, a hub-level fact like `fabric.yaml`'s pathspec.
The rest tune the run in the worktree that reads them (models, timeouts, review budgets, reed's per-worktree watchdog, shuttle's worktree-local run dir) or are read only from the prime (`orch.yaml`), so a per-worktree file is correct for them, and a repo-wide default for those is `config-overlay`'s job.

### D2 — The hub-wide set is declared once, in `configreg`

Decision: `configreg.Module` gains a field marking a module hub-wide, set for `fabric` and `board`.
Every site that today special-cases `"fabric"` by name (`configsync.ReconcileAll`'s skip, the fabric-only reconcile) branches on that field instead.
Rationale: one declaration keeps `configsync`, `configcli` and any later hub-wide module from each spelling their own list.

### D3 — Board config is loaded from the board dir

Decision: `boardengine.LoadConfig` keeps its told-`baseDir` signature, and every caller passes `fabricengine.BoardDir(hub)` instead of the worktree anchor: `boardcli`'s `PersistentPreRunE`, `loomcli`'s `openHubBoard` and `ideengine.Menu`.
The detached sync child (`--board-path`) is unchanged; it loads no config.
Loading stays strict (`configengine.Load`, per `PATTERN-config-strictness`): an absent hub `board.yaml` is refused with the existing "not initialized here; run `lyx fabric reconcile`" way forward.
Rationale: the engine stays told its geometry (`PATTERN-told-geometry`); only CLI-level callers, which already resolve the hub through `lyxcwd`, change what they pass.

### D4 — Hub-wide reconcile seeds `board.yaml` from the prime's copy

Decision: the fabric-only `configsync.ReconcileFabricAt` becomes a reconcile of every hub-wide module at the board dir, run where it runs today (`lyx fabric clone`, `lyx fabric reconcile`), carrying each module's open maps (`board`'s `types` and `labels`) whole.
When the hub `board.yaml` is absent, it is seeded from the prime worktree's `_lyx/config/board.yaml` if that file exists, else from the template, and then reconciled against the template; `fabric.yaml`'s existing legacy warp/weft fold-in is kept.
A present hub file is never replaced by a seed; reconcile only adds and prunes template keys, as today.
The prime's path is resolved by the CLI caller and told to `configsync`.
Rationale: the prime's copy holds the hub's live vocabulary (the `feature` type that triggered this bug); seeding from a pair's copy would install exactly the stale vocabulary this task removes.
Bound: the seed reads one file, only when the hub file is absent, and only from the prime.

### D5 — `lyx config` addresses a hub-wide module at the board dir and commits it there

Decision: `lyx config <module>`, `--set`, `--print` (single module and the all-modules dump) and `lyx config menu` resolve a hub-wide module's file at the board dir.
After a successful edit or `--set` of a hub-wide module, the change is committed and pushed in `_board` through `fabricengine.Bolt`, under the board's write lock as board sync does, instead of the worktree's fabric sync.
Error and success messages name the file actually edited.
This also fixes `lyx config fabric`, which today edits a per-worktree `fabric.yaml` nothing reads.
Rationale: an edit that lands in `_board` but stays uncommitted would be swept into an unrelated "board sync" commit or lost on a fresh clone.
Bound: any worktree can now change the hub's board vocabulary through `lyx config board`; that is the same trust boundary as `lyx board upsert`, which already writes the hub board from any pair.

### D6 — `lyx config reconcile` retires the per-worktree copy

Decision: `configsync.ReconcileAll` skips hub-wide modules for reconciliation, and when a worktree holds a per-worktree file of a hub-wide module while the hub file exists, it reports that file as removable and deletes it on `--apply`.
It never deletes a per-worktree copy while the hub file is absent.
The result for such a module names the removal, so `lyx config reconcile`'s JSON shows it.
Rationale: a dead `_lyx/config/board.yaml` left in every weft invites hand edits that silently do nothing, as `board-model`'s D13 label list did.
Bound: deletion is limited to `_lyx/config/<module>.yaml` of a module the registry marks hub-wide, under `--apply`, with the hub copy present.

### D7 — A PATTERN entry states the rule

Decision: add `PATTERN-hub-wide-config` under "Paths and geometry": a config module describing a hub-level fact is marked hub-wide in `configreg` and read and written only at `<BoardDir>/_lyx/config/`, never from a worktree's `_lyx/config/`.
It lands in the same commit as the code, without a background file unless the plan finds the rationale does not fit the one-line entry.
Rationale: CLAUDE.md requires a new cross-cutting invariant to land in PATTERN with its code.

## Constraints

- `PATTERN-told-geometry`: `boardengine` and `configsync` take absolute paths; only CLI packages resolve hub, board dir and prime.
- `PATTERN-cwd-resolution`: hub and prime resolution go through `internal/lyxcwd` (and `fabricengine` listing for the prime), never `os.Getwd` or `git rev-parse`.
- `PATTERN-lyxdirs-single-declarer`: board-dir config paths are built with `configengine.ConfigFile` / `lyxdirs`, never a spelled `_lyx`.
- `PATTERN-hub-containment`: `_board` is reached by resolved hub path; nothing is junctioned into a worktree.
- `PATTERN-fabric-git`: the `_board` commit and push go through `fabricengine.Bolt`, never raw git.
- `PATTERN-config-strictness`: board config loading stays on `configengine.Load`.
- `PATTERN-cliwire-sole-wiring` and `PATTERN-cli-cobra`: CLI wiring changes stay inside the existing descriptors and cobra subtrees.
- `PATTERN-test-economy`, `PATTERN-test-tier-purity`, `PATTERN-hubforge-fixtures`, `PATTERN-hermetic-git-tests`: tests that need a real hub use `hubforge`, under the `integration` tag; untagged tests stay git-free.
- `PATTERN-comment-line-breaks` and semantic line breaks in every `.md`.
- Docs in the same commit: `internal/boardcli/doc.go`, `internal/boardengine/doc.go`, `internal/configcli/doc.go`, the `configsync` and `configreg` package docs, `docs/overview.md` and `docs/shared-libs/configengine.md` where they place `board.yaml` per worktree.

## Auto-mode assumptions

- No operator answered; every choice in Decisions is an auto-pick logged in the support log's question ledger.
- The prime is the worktree checked out on the warp default branch, resolvable from the hub through the existing worktree listing.
- `internal/boardengine/doc.go`'s "loomyard's own area labels" paragraph tells the hub to write a label list into the parent's live per-worktree `board.yaml`; the plan rewrites it to point at the hub file, or drops it if the list is already live on the hub.

## Open risks

- A hub where `lyx fabric reconcile` runs from a pair before the prime has a hub `board.yaml` still seeds from the prime, but a hub without a resolvable prime falls back to the template and loses its custom vocabulary until the operator edits the hub file; the reconcile result reports which source seeded it.
- Deployment order: the landed binary refuses `lyx board` on a hub until `lyx fabric reconcile` has seeded the hub file, so the landing's deploy must be followed by one `lyx fabric reconcile`; the PR description states this.
- Pairs running an older binary keep reading their own copy until they are torn down; the fix takes effect per binary, not per worktree.

## Acceptance criteria

- `lyx board` in a pair whose own `_lyx/config/board.yaml` lacks a type that the hub file has renders README with that type, and accepts a label the hub file lists.
- `lyx board`, loom's landing board access and `lyx ide` menu all read the hub file; a per-worktree `board.yaml` alone does not satisfy them.
- An absent hub `board.yaml` makes a board verb refuse with a message naming `lyx fabric reconcile`.
- `lyx fabric reconcile` with no hub `board.yaml` seeds it from the prime's per-worktree copy when one exists, else from the template, carries `types` and `labels` whole, and commits it in `_board`; with a hub file present it never replaces it.
- `lyx config board --set labels.x=...` and `lyx config fabric` edits from a pair change the hub file and leave a commit in `_board`; `--print` shows the hub file.
- `lyx config reconcile` neither reconciles nor writes a per-worktree `board.yaml` or `fabric.yaml`; with `--apply` and a hub copy present it deletes the per-worktree copy and reports it, and without a hub copy it leaves it.
- `configsync` and `configcli` decide hub-wide handling from the registry field; no site branches on the literal module name `"fabric"` or `"board"` for this.
- `PATTERN.md` carries `PATTERN-hub-wide-config`, and the listed docs no longer place `board.yaml` per worktree.
- `go test ./...` and the `integration` tier pass.

## Notes for the plan writer

- Current special-casing of `fabric` lives in `configsync.ReconcileAll` and `configsync.ReconcileFabricAt`; callers of the latter are in `internal/fabriccli` (`clone.go`, `fabric.go`'s reconcile handler).
- `configcli` computes one `baseDirOf(l)` for every path; hub-wide modules need the board dir instead, and its sync seam (`syncFunc`) is where the `_board` commit replaces fabric sync.
- `boardengine.Sync` already commits and pushes `_board` through `Bolt` under the board's write lock; reusing it, or its lock-plus-commit step, is a likely route for D5.
