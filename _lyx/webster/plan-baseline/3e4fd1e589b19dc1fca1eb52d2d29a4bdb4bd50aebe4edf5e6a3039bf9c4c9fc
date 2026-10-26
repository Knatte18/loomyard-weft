---
format: 5
approved: true
language: go
---

# Plan: beam-start-and-ownership

A beam, the prime included, becomes one unit of two parts, its dobby and its orch, that run on the machine owning it.
`lyx beam start` brings a beam up in place of `lyx orch start`, ownership moves from tasks to root and beam projects, `lyx adopt` moves a project, and a beam's whole life is one batten run over its project entry with the `pawl` recipe: Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize and Beam-Down.
The cards first remove what goes (`lyx beam promote`, the task owner, the orch's start and cleanup verbs), then build the project owner, the landing hold and the named dobby starts, then the orch and beam verbs, then the pawl recipe and its wiring, then batten, create, land, dobby and adopt over it, and last the wording every way forward carries.

## Card Index

1 — remove-beam-promote — remove `lyx beam promote` and every helper and way forward only it needed
2 — end-processes-by-role — `runproc.EndRole` ends one role's records under an owner
3 — owners-on-projects — owners on root and beam projects only; a task reads its beam's or root's owner
4 — adopt-projects — `lyx adopt <beam>...` and `lyx adopt root` move a project and the runs under it
5 — landing-hold — `RunHolds`: a pawl run holds its beam only from Parent-Align on
6 — recipe-named-pawl — the `beam` recipe becomes `pawl`, and a `beam` seed reads as `pawl`
7 — dobby-start-named-beams — `EnsureLive` starts only named beams and skips a held one
8 — orch-watcher-owner — the orch's watcher stops the orch of a beam another machine owns
9 — remove-orch-cleanup — remove `lyx orch cleanup`
10 — orch-beam-up — `orchcli.BeamUp` and `BeamDown` replace `lyx orch start`, with resume-else-note-else-stencil
11 — beam-start-stop-status — `lyx beam start [slug] [--all]`, `lyx beam stop`, and the orch in `lyx beam status`
12 — shortcut-s — add `lyx s`, drop `lyx prime`
13 — land-blockers — `BeamLandRefusal` judges members and pairs alone, readable as blockers
14 — finalize-land-recheck — Finalize re-checks a beam's members under the parent lock
15 — pawl-engine — the Beam-Up, Beam-Await and Beam-Down producers and the land request
16 — pawl-recipe-rows — the pawl recipe's nine rows, their registry entries, budgets and profiles
17 — pawl-wiring — wire the pawl rows' seams and Finalize's re-check in loomcli
18 — batten-beam-project — batten starts a beam project's run and removes a member forked under a hold
19 — batten-quiet-at-beam-await — no quiet notice for a child at Beam-Await
20 — batten-cancel-beam — a beam's cancel reconciles, refuses while members remain, and stops its orch
21 — beam-create-starts-pawl — `lyx beam create` records the beam and starts its pawl run
22 — beam-land-request — `lyx beam land` writes a land request Beam-Await consumes
23 — dobby-land-notice-and-slots — the "can land" notice, and no slot for a pawl run without its hold
24 — adopt-beam-steps — adopt brings up each adopted beam's dobby and orch
25 — adopt-subject-ways-forward — owner ways forward name the beam or `root`, never a task
26 — name-beam-start-everywhere — every way forward names `lyx beam start` where it named `lyx orch start`

## Shared Decisions

- `boardengine` imports no recipe package, so it declares `ProjectRecipe` (`pawl`) and the hold-free producers itself; `TestBoardRecipe` (card 6) pins `ProjectRecipe` to `shedrun.RecipePawl`, and `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` (card 16) pins `RunHolds` to batten's and pawl's row names.
- `Owners.AdoptSubject` is the one place that names what `lyx adopt` takes for an entry: the beam project itself, else its nearest beam, else `root`; every adopt way forward reads it through the `ParentOf` resolver its caller already holds (`PATTERN-pair-parent`).
- The beam steps live in `orchcli` (`BeamUp`, `BeamDown`), since orchcli already holds the orch start, the beam window and the dobby start and every caller (beamcli, adoptcli, loomcli, battencli) may import it; `Prepare` alone runs the reconcile, the root claim and the owner pauses, so adopt and Beam-Up, which already judged origin, skip them.
- A resumed orch session is launched with the reload pointer, the one-line "read the role file and continue" stencil, since the adopt stencil goes with `--adopt`.
- The pawl run's seed id, which a land request names, is its child seed's `started_at`: a re-created pawl run writes a new seed with a new one, so a request left from an earlier run is dropped.
- Pawl's Finalize declares no `on_reroute` (D8), so its `RerouteReviewed` seam stays nil and a spent or over-cap fixer halts for the orch; its perch carries no `carry_over`, since a pawl run has no decision record to file open findings into.
- The land request, the dobby-stop marker and Beam-Await's await marker sit under the beam worktree's `.lyx/beam/` beside the dobby's files, composed by `hubgeom.PawlPaths` from `lyxdirs`, so dobby, `lyx beam land` and Beam-Await read one directory.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run: only where its entry is not owned elsewhere; owners only on root and beam projects, set only by `boardengine`'s owner setter; any other entry reads its `hubgeom.ParentOf` owner; a dobby continue pass is no resume. — [background](pattern/PATTERN-owner-machine.md)
- `PATTERN-run-process-owner` — changes: A run's processes: registered in `internal/runproc`, ended only via `runproc`'s `End`, `EndSparing` or `EndRole`.

## Rename mechanic

1. For a file pair (both sides file self glyphs or paths), run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.
5. For a symbol pair, run the hub's structure-aware rename tool the implementer prompt names (`language.yaml` `rename_tools`), never a text or regex replace, then grep for zero remaining old-name references and confirm the symbol's doc comment opening names the new name.
