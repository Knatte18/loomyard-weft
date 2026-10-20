# Round 3 fixer report — discussion

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  D6 states that a `generation_start` Stuck skips Shed's budget comparison, as a `BudgetExempt` Stuck does, while it is recorded as counted, so it never takes `budget_capped` and a seed over a spent episode starts the generation.
  The Scope line on `generation_start` names the skip.
  A criterion covers an interrupted settle at the perch's last round followed by a changed subject.
  A plan-writer note names the `!output.BudgetExempt && episodeStuckCount(...) >= effectiveMaxBounces(...)` case of `stepLocked` in `internal/shedengine/run.go`.
- **F2 (MEDIUM), fixed as proposed (first alternative).**
  D5's settle record stores the full working-tree diff as a patch file beside it.
  D6's code base is the record's merge base and stored patch; the settled side is the stored diff the settle hashed and the current side is the working-tree diff against the current merge base, untracked files outside the exclusion set included, so an uncommitted change at the settle is never shown as new.
- **F3 (MEDIUM), fixed as proposed.**
  D6's document base is the weft commit that holds the settled artifact, read by a told seam after the Commit seam returns and written when the record is marked settled; an interrupted settle keeps the weft HEAD read with the identity, and its amendment reviews the change since that commit, the whole uncommitted artifact when no earlier commit holds it.
  D5 says the record is written with that HEAD and that marking it settled replaces it.
  A criterion covers a Plan-Bouncer amendment after a PR-Gate reject being told a base that contains the settled plan.
- **F4 (LOW), fixed as proposed.**
  D5's untracked clause has an exclusion set: the run's own state, as `UncommittedPaths` excludes it, plus a told list of harness files, `.scratch_task.json` among them.
  A criterion covers a file outside and one inside the set.
- **F5 (LOW), fixed as proposed, extended.**
  The Merge-In test reads first-parent commits since the settle commit, and a commit came from a Merge-In when it is a merge whose second parent is an ancestor of the recorded parent branch's tip, or is itself such an ancestor, which covers a fast-forward Merge-In (`internal/fabricengine/merge.go:116`).
- **F6 (LOW), fixed as proposed, adjusted per recipe.**
  D5 names what the patch-id rule lets through, a parent change interacting with the task's change without sharing a line, and what bounds it: the verify tiers, at Publish in loom and at darn's Merge-In, and loom's Merge-Review segment unless its skip seam or a zero budget settles it.
  The proposed "Merge-Review segment" bound alone was not written, since darn has no Merge-Review perch (`contracts/recipes/darn-recipe.yaml:178-179`).
- **F7 (LOW), fixed as proposed.**
  Q13 states the current D5 form, and the support log has Round 2 and Round 3 entries.

## Self-check

- fix-consistency, MEDIUM: an interrupted settle's document base is never stored, since D5 wrote the base only at marking settled — fixed as proposed (record written with the weft HEAD read with the identity, replaced at marking).
- fix-consistency, LOW: the stored patch is missing from the archive sentence and the `PATTERN-tolerant-records` constraint — fixed as proposed.
- fix-way-forward, MEDIUM: interrupted settle's document base not stored — duplicate of fix-consistency's MEDIUM, fixed.
- fix-claims, MEDIUM: the patch-id bound names Merge-Review, which darn lacks, and loom's Merge-Review can skip — fixed as proposed (bound stated per recipe).
- fix-claims, note: a fast-forward Merge-In leaves non-merge first-parent commits — fixed as proposed (an ancestor of the parent tip counts as from a Merge-In).
- fix-proposal, MEDIUM: same darn bound — duplicate of fix-claims, fixed, with darn's Merge-In verify of the merged tree named.
- fix-proposal, MEDIUM: archive drops the stored patch — duplicate of fix-consistency's LOW, fixed.
- fix-proposal, LOW: interrupted settle's base — duplicate, fixed.
- fix-proposal, LOW: a task commit of hashed uncommitted work followed by a Merge-In reads as changed — fixed as proposed (first alternative: D5 names it an accepted false amendment bounded by the allowance).

## Disputed

None.

## Deferred

None.
