---
format: 5
approved: false
language: go
---

# Plan: beam groundwork

Give the hub beams: long-lived pairs that group board work, spawn their tasks from their own branch and take them back, each with an optional orch of its own.
The plan builds fabric's parent fork, adopt-local add, beam mark, per-beam workspace files and records reset; the board's project kind, membership, beam flag, parent resolver, slug rule and README section; hubgeom's one `ParentOf` and its live-orch walk; orch and batten running from any beam, with notices routed up the beam chain; the `beam` landing recipe; and the `lyx beam` module with `create`, `promote` and `land`.

## Card Index

1 — fabric-slug-rule — export fabric's pair slug rule for the board
2 — origin-beam-mark — add the beam mark to the origin record and its writer
3 — add-options-struct — make AddOptions a struct of its own
4 — add-from-parent — fork a pair from a named parent pair
5 — add-adopt-local — make a pair over an existing local branch
6 — fabric-add-flags — expose --parent and --adopt-local on lyx fabric add
7 — beam-workspace-files — write a workspace file per beam beside the hub's
8 — weft-reset-executor — reset a pair's records content to its parent's
9 — board-project-kind — add the project kind with its fields, handle and layer
10 — board-membership — judge project and order, and carry them through rename, prune and merge
11 — board-beam-flag — write the beam flag only through the owner setter
12 — board-parent-beam — resolve an entry's beam from the project chain
13 — board-slug-seam — apply fabric's slug rule to task and project slugs
14 — board-readme-projects — render the Projects section of the README
15 — hubgeom-parent-of — one parent resolver for every owner reader
16 — delete-parent-slug — remove fabric's branch-matching parent read
17 — boardcli-wiring — wire the slug rule, pair presence and parent_beam into lyx board
18 — hubgeom-live-orch-walk — walk to the nearest live orch up the beam chain
19 — orch-in-beam — run lyx orch in the prime and in any beam
20 — orch-notify-parent — notify a run's nearest live orch
21 — tender-every-orch — make tender, hibernate and the watcher read every orch
22 — overview-cleanup-every-orch — make the overview and cleanup read every orch
23 — batten-guard — admit batten, adopt and cleanup from any beam
24 — batten-window-site — open a run's window in its parent's session
25 — batten-notices-walk — route batten's notices up the run's beam chain
26 — land-ready-routing — name a ready task only to an orch that may start it
27 — batten-cleanup-skip — name the cancel in cleanup's not-done skip
28 — worktree-create-from-beam — fork a beam member's pair from the beam
29 — finalize-large-merge-lands — let a Finalize gate a large merge instead of rerouting
30 — beam-describe-producer — write a beam's description from its entry and commits
31 — beam-recipe — add the beam recipe and register its row
32 — loomcli-beam-recipe — run the beam recipe through loom's bootstrap
33 — batten-seed-beam — seed the beam recipe for a project entry
34 — beam-create — add the lyx beam module and its create verb
35 — beam-promote — turn a spawned task into a beam
36 — beam-land — land a beam through batten and the beam recipe
37 — adopt-beam — let lyx adopt take over a beam

## Shared Decisions

### Decision: beam-detection

- **Decision:** a worktree is a beam when its own origin record carries the beam mark; no reader derives it from the board.
- **Rationale:** the origin record travels with the records branch, so every machine reads the same answer without fabric or batten reading the board (D8, D12).
- **Applies to:** cards 7, 19, 21, 23, 35, 36

### Decision: parent-reads

- **Decision:** a new pair's parent comes from `ParentBeam`, an entry's parent for its owner and `parent_beam` from `hubgeom.ParentOf`, and a notice's destination from `hubgeom.NearestLiveOrch`; `hubgeom.ResolveParent` keeps answering the parent pair itself.
- **Rationale:** D4 and D13 name these three reads and keep them apart.
- **Applies to:** cards 12, 15, 18, 20, 25, 26, 28

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd or worktree root: only `internal/lyxcwd`; an engine is handed absolute paths and never imports it; hub-level containers and the hub's workspace files stay reachable from the hub only.
- `PATTERN-pair-parent` — adds: Picking a new pair's parent for a board entry or reading an entry's owner through its parent: only `ParentBeam` and hubgeom's one `ParentOf`, never the acting worktree.
