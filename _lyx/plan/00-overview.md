---
format: 5
approved: false
language: go
---

# Plan: Finalize names the dirty parent checkout and its paths

When Finalize's parent-side merge refuses on a dirty parent checkout, its stuck reason names the parent's code checkout and the tracked dirty paths the guard decided on, with a way forward per reason.
No goto onto Finalize, and no Go driver starting at a blocked Finalize stop, re-steps Finalize into the same refusal, and the halt note drops the separate parent read that counted lyx's own run records as dirt.

## Card Index

1 — guard-error-carries-dirt — the dirty merge guard decides on its path list and the guard error carries the checkout and the paths outside `Error()`
2 — parent-reads — `Fabric.CodeDirtyPaths` and `Fabric.MidMerge`, the read-only parent reads the later cards use
3 — finalize-dirty-reason — Finalize's dirty-parent stuck reason per reason, the shared renderer, the refusal-spec amendment and the run-record criterion
4 — halt-note-drops-parent-status — the halt note loses its `parent fabric status:` section and the parent read behind it
5 — finalize-goto-guard — the goto guard refuses a goto onto Finalize while the parent's code checkout is dirty
6 — driver-start-check — a Go driver starting at a blocked stop runs the Finalize arm before its first step

## Shared Decisions

- One renderer, `landingshed.DirtyParentRefusal`, words the checkout, the path list and the dirt-only way forward for Finalize (card 3), the goto guard (card 5) and the driver's start check (card 6), so the three never drift apart.
- Finalize reads the parent pair's parked merge through the `parentMerger` handle it already holds (`Fabric.MidMerge`), so `landingshed.Deps` gains no field.
- D2's merge-in-progress way forward tells the operator to commit or set aside their own change with plain git before `--abort`; card 3 words it as saving a copy outside the checkout instead, because a plain `git commit` in a checkout holding a staged fabric merge concludes that merge, after which `--abort` refuses with `merge conclude already landed` (`sideConcludeMayHaveLanded`), and a `git stash` there takes the staged merge with it.
