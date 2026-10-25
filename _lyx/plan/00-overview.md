---
format: 5
approved: true
language: go
---

# Plan: umbrella-anchor — an anchor per beam

Each worktree's anchor moves from the one Bolt marker to a `.lyx-anchor` at the root of its own weft branch, read by `lyxcwd` per worktree and inherited through git by every fork.
Every path lyx builds for a worktree uses that worktree's own anchor, `lyx fabric reconcile` migrates markerless wefts and retires the Bolt marker, every weft writer stops rather than cross a marker change, and `lyx fabric anchor` and `lyx beam create --anchor` move a worktree's anchor with recovery from a kill.

## Card Index

1 — lyxcwd-own-marker — lyxcwd reads each worktree's own marker, NamedWorktree, PATTERN line
2 — location-literal-scan — replace the production Location literals and scan for them
3 — fabric-target-locations — fabricengine builds another pair's paths at that pair's anchor
4 — consumer-target-locations — dobby geometry, teardown, IDE and reconcile CLI use each worktree's own anchor
5 — exclude-census-per-anchor — the exclude census keys on the anchored pattern
6 — clone-records-marker-on-prime-weft — clone records and adopts the marker on main-weft
7 — add-takes-the-fork-point-anchor — Add forks at the fork point's anchor and refuses an empty fork-point marker
8 — reconcile-migrates-markers — per-pair marker checks, migration, D5's value and the Bolt census
9 — weft-pull-and-push-stop-on-marker-change — the one comparison, the pull and fast-forward stops and the replaying-push stop
10 — checkout-and-drop-local-refuse-another-anchor — checkout and drop-local refusals
11 — origin-reconcile-stops-on-marker-change — the origin reconcile leaves a weft unmoved as anchor_stopped
12 — promote-refuses-another-anchor — beam promote refuses a task at another anchor than its parent's records
13 — move-record-stops-weft-writers — the move record and lock, and the refusals a standing record raises
14 — weft-integration-across-anchors — the integration that carries a weft across an anchor change
15 — anchor-change-engine — fabricengine's move, integration step and recovery
16 — dobbies-wait-out-a-move — dobbies start no pass and EnsureLive starts no dobby while a record stands
17 — fabric-anchor-verb — lyx fabric anchor [<slug>] [--set <dir>]
18 — beam-create-at-an-anchor — lyx beam create --anchor <dir>
19 — adopt-refuses-during-a-move — lyx adopt refuses while a move record stands

## Shared Decisions

- **Another worktree's Location.** Production code outside `lyxcwd` gets another worktree's Location only from `lyxcwd.NamedWorktree(hubPath, name)` (card 1), or `ResolveWorktree` from a worktree root it holds; `ResolveWithAnchor` keeps its two callers plus `Add`'s new pair at a given anchor, whose warp worktree exists by then (card 18).
- **The `(l, slug)` builders keep their signatures.** D3 says they take the target worktree's Location instead of the acting one's `AnchorRel`: a pure path builder (`WarpJunctions`, `CodeLyxLink`, `WeftLyxDirFor`, `LauncherDir`, `OriginRecordPathFor`, `scratchLinkTarget` and the like) takes the target's Location as its `l`, and every caller acting on another worktree passes that Location; a function that already returns an error and takes a slug (`ReadOriginFor`, `WriteOrigin`, `MarkBeam`, the remove helpers) reads the target's Location itself through `NamedWorktree`.
  Re-signing the builders would touch some hundred test call sites in one-anchor hubs for no behavior change, and the review-held rule of D3 covers a builder handed a mismatched `l`.
- **Who writes a marker.** Only `fabricengine`: `commitAnchorMarker` (clone's fresh bind, reconcile's migration, Add's adopt of a markerless branch) and the D7 move commit, `moveWeftContent` (card 15's move, and card 18's create at an anchor); both write the file through an `os.Root` at the weft root. `committedMarker` and `treeAnchor` are its readers of git objects; `lyxcwd` reads only the working tree.
- **Fixtures for a second anchor.** Until card 15 exists, tests make a pair at another anchor with `hubforge.ReanchorPair` (card 3), which commits the marker and the moved content in the pair's weft and rewires the pair; no test writes a weft marker without committing it, since an uncommitted marker is weft dirt several verbs refuse.
- **The engine and CLI split of D7.** `fabricengine` exposes `BeginAnchorChange`, `Apply`, `Finish` and `Close` (card 15); `fabriccli` (card 17) runs the board, reed, run, gate-slot, hibernate and dobby checks, the pass wait, the parent lock, the dobby end and restart and the log sink around them, as `pairteardown` sits above fabric.
- **Stops share one state and one error.** A weft stopped by a marker change, an empty own marker or a standing move record is `ErrAnchorChange` on the pull and push paths, a landing's fast-forward weft pull included, which skips the weft and goes on with the code side (an `*AnchorStopError` carrying its way forward on the push, recorded as the failed-push class `anchor_change`) and `BranchAnchorStopped` in the origin reconcile, each with the way forward the record names.
- **Every way forward progresses.** A stop names the `--set`; a `--set` refused for an unfinished run names `lyx batten cancel`, which an entry stopped only by `anchor_stopped` passes, and for a worktree another machine owns judges its own run from the weft's fetched upstream tip and names the cancel on the owner machine, then `lyx fabric pull` to fetch that tip; the owner refusal also names the `--set` to the fetched upstream tip's marker, which it lets through; a re-run recovering a standing move record skips the owner refusal; a failure before anything moved leaves no record and restarts the ended dobbies, and every failure that leaves the record, an `Apply` failure after the move landed or a failed restore included, starts no dobby and names that re-run; promote checks the anchor only after its own steps have ended and cleared the run.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `internal/lyxcwd`, from the worktree's own weft-root `.lyx-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lyxcwd; hub containers and workspace files stay hub-reachable only. (test)
