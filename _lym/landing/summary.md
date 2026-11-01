# Finalize settles a rejected fixer change in place, never rerouting

Finalize no longer sends a run back to an earlier row.
Before this change, a merge fixer that spent its cap, or made a change over the caps that Finalize's fix review did not approve, rerouted the run: tapestry to Parent-Align, darn and sampler to Code-Bouncer.
Pawl, with no reroute edge, halted instead.
Now every recipe settles that change inside Finalize, under the parent lock.

## What changes

- **The fix review loop.**
  A fixer change over the caps goes to the fix review in every recipe, pawl included.
  A review that does not approve writes its findings into the attempt record, and Finalize spawns the merge fixer again in finalize mode with the review report as `fix_review_path`, on a fresh `finalize_fix_attempts` turn cap.
  The fixer stencil tells it to fix each finding while keeping every test's meaning.
  Its fixed change is judged against the caps again: within them it lands, and over them the review runs again.
  A review that gives no verdict runs again with no fixer turn.
  Each review spends one of the new `landing.yaml` key `finalize_fix_review_rounds` (default 3, refused below 1).
- **The cap halt.**
  When the last round's review does not approve, Finalize halts with `LandingStopPrefix`.
  The attempt record notes `review-capped` and the open findings.
  The reason names the findings, the fixer commits, and two ways forward: `lym loom circling accept <slug>`, or fixing the findings on the task branch, each followed by `lym loom resume`.
- **Acceptance.**
  `circling accept` on a run blocked at Finalize calls the new `landingshed.AcceptFixReview`.
  It appends a `## Unresolved fix review findings` section to the change description and binds the acceptance to the task HEAD in the attempt record; the CLI commits the description.
  On resume Finalize lands the change at that HEAD as it stands, and any later change goes to the fix review.
  A resume without acceptance clears the round count and runs the fix reviews again.
  `circling continue` at Finalize is refused, and its way forward names both routes.
- **Spent cap.**
  A fixer that spends its cap over a still failing gate halts with `LandingStopPrefix` and `spent` noted, as pawl's did before, since nothing can land over a failing gate.
- **Removed.**
  `on_reroute` is gone from Finalize in the tapestry, darn and sampler recipes.
  So are `landingshed.Deps.RerouteReviewed`, its loomcli wiring, `Finalize.reroute` and `rerouteReason`, and the reroute-cap clause of `landingshed.HandsBack`.
  `loomRerouteCapWayForward` keeps only the Webster plan-conflict clause, since Webster is the one loom row left with a reroute edge.
  The over-cap halt Finalize used on pawl, with its Baseline rewrite on resume, is replaced by the review loop.

Docs follow the code: the landingshed and loomrecipe package docs, `docs/overview.md`, the recipe comments, `landing-fix-review-spec.md`, `loom-status-spec.md`, `loom-plan-spec.md` and the orch role stencil's landing-halt guidance.

## Design decisions

- **The cap halt goes to the orch, not the arbiter.**
  It opens with `LandingStopPrefix`, so `HandsBack` routes it to the orch like every other landing halt and the operator decides through it.
  The fixer's escalate halt keeps `EscalateStopPrefix`.
- **Acceptance reuses `lym loom circling accept`, with no new verb.**
  This mirrors how a review loop out of rounds is accepted.
  The verb branches on a run blocked at Finalize before its Bouncer checks.
  `AcceptFixReview` is idempotent: a description that already holds the section is not appended to again, and an entry already accepted writes nothing, so a retry after a failed commit is safe.
- **Each review round gives the fixer a fresh turn cap.**
  The reset is written in the same record write that counts the round, so a resume never gets an extra cap.
  The loop is bounded by `finalize_fix_review_rounds` × `finalize_fix_attempts` fixer turns.
- **A spent fixer still halts.**
  A spent cap leaves a failing gate that no review can approve, so it stays a halt for the operator, now in every recipe.

## To check by hand

- A run in flight whose Finalize record notes the old `over-cap` halt resumes into the fix review loop, not the old acceptance-on-resume.
- `circling accept` at Finalize trusts the attempt record in the scratch directory.
  A fresh checkout of the task branch has no such record, so it refuses there with the not-halted way forward.
- The loop was not run end to end against a live fixer and reviewer; only the package tests exercise it.
