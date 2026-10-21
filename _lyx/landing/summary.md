# Goto, resume and stops keep what a run already settled

A goto, a resume or a stop no longer discards a converged review verdict, the spent review budget or the reason a run paused.
A review segment re-entered after its verdict settled reviews only the task's change since then, under a small allowance, or reviews nothing when nothing changed.
The motivating run was `ubuntu-2404-support`, where one wrong goto cost a fresh review round over an approved plan, a friction session per stop and a paused state nobody could explain.

Closes #759
Closes #756

## What changes

- **Goto keeps the spent budget.**
  `episodeStuckCount` in `internal/shedengine` counts across a goto entry; a reroute into a segment still ends its episode.
  A goto still resets what the target row needs to run.
- **Goto never re-runs a settled review.**
  A goto onto a review or fix row of a segment with a live settle record is refused with the status file unchanged; the way forward names the Bouncer row and the writer row.
  Loom's verify retry onto the row its Publish failure record names is the one admitted case, until that generation settles again.
  The check is a told seam on `GotoRequest` and `RelaunchRequest`, filled by loom and darn, and the tackler and arbiter run it when they validate a verdict's goto.
- **A budget-exhausted halt is lifted by the parent's resume only.**
  The halt's Stuck entry is marked `budget_capped`, its way forward names the recipe's resume verb, and `internal/sheddriver` hands it back to the parent before any tackler or arbiter rule.
  One resume grants one more budget; a goto grants none.
- **Every write of `paused` records `pause_reason`.**
  The pause verbs take a visible `--by` caller label, which refuses `shed`.
  `lyx loom status`, `lyx shed status` and the drive report show the reason, and a paused status with none is labelled as having no recorded reason.
- **The Bouncer records its settle and compares on re-entry.**
  An unchanged subject re-settles at no review cost, which also makes a failed Commit after a converged verdict re-run only the mechanical step.
  A changed subject starts an amendment generation, scoped to the change since the settled base, under `review_amend_max_bounces` (default 1, 0 settles with no review).
  Merge-Review takes no amendment: a changed merge-in starts a full generation.
  PR-Rework's archive keeps the settle, generation and patch records, so a PR-Gate reject reaches the same path.
- **No friction session per stop.**
  A `blocked` or `failed` halt still writes its `loom-halt` note and starts no reflection; the Friction-Reflect row reflects it at the run's end.
- **A review round with the wrong fork count is retried** once, and the failure names the requested and spawned counts, the chair's strand and the session states shuttle recorded.
- Help texts, refusals and stencils no longer name a goto as the way to a fresh budget.

## Design decisions

- A code segment's identity is the diff of the warp working tree against the task branch's merge base, read through `internal/fabricengine` (new patch reader).
  It is unchanged when the merge base and diff hash match, or when only Merge-Ins moved the base and the zero-context patch-id matches.
  A parent change that interacts with the task's change without sharing a line with it passes unreviewed by that segment; Publish and darn's Merge-In verify tiers bound it.
- A task commit after the settle reads as a change even when it commits work the settle already hashed; the false amendment costs at most the allowance.
- A standalone run records no parent branch, so its code segments write no settle record and take today's clear-and-reseed.
- A run directory or status file from an older binary keeps working; an in-flight run has no settle records, so its first re-entry re-reviews in full.
- A budget-exhausted halt an older binary wrote has no `budget_capped` entry, so neither its resume nor a goto grants a fresh budget.
- `PATTERN-friction-capture` changes with the removal of per-stop reflection.

## To check by hand

Not run, or left open by review:

- Id `card12-start-generation-name-collision` (consistency, blocking when raised): check that the amendment generation start does not collide with an existing generation-start name in `internal/shedadapters` or `internal/shedengine`.
- `new-keys-break-exact-set-pins-and-docs`: check that every exact-set pin and doc listing review budget keys includes `review_amend_max_bounces`, in `reviewbudget`, `loom.yaml` and `darn.yaml` templates and their docs.
- `nillegal-hides-missing-reject-fill-at-wiring`: check that a nil `Legal` seam in the loom and darn wiring cannot hide a missing reject-file fill.
- `rewritten-sentence-carrier-not-listed-glyph`: check the rewritten rubric and help sentences for a carrier that is not a listed glyph.
- `undeclared-pattern-overview-edits`: check that the `PATTERN.md` overview edits are only the declared `PATTERN-friction-capture` change; the shortened friction-capture line and the restored push-both-sides and leaf-packages lines are the places to look.
- The Discussion finding `d3-goto-after-halt-grants-budget`: confirm a goto onto a budget-exhausted halted row leaves its count at the spent budget.
- The webster verify gate's first attempt failed because the untracked `.scratch_task.json` made the worktree dirty; confirm the harness file stays in the code identity's exclusion list.
- After landing, the hub orch rule "A review segment that already converged" is removed through `lyx orch rules`.
