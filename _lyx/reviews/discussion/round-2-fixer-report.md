# Round 2 fixer report — discussion

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  D6's Merge-Review sentence now has case 2 record the next generation number, with no amendment mark and no base, before its seed, so the kept settle record names an older generation and is no longer live.
  The Merge-Review criterion now requires that the new generation's second Call judges round 1 without a second archive.
- **F2 (MEDIUM), fixed as proposed.**
  D5 keeps one settle time, redefined: the time the record was written by a settle on the round-by-round path (a round N+1 after the verify retry or the Webster reroute included) or an `approveWithoutReview` path; a case 1 re-settle keeps the stored time.
  D2 names the comparison as the failure record's write time against the live record's settle time, and says a case 1 re-settle after a Publish failure leaves the bypass open.
  A criterion pins that window: after the failure record is written and before the retry runs.
- **F3 (MEDIUM), fixed as proposed (first alternative).**
  D6: `episodeStuckCount` counts the `generation_start` entry and stops after it, so the count reads 1 once the seed is recorded.
  D6's bound and its criterion name the rounds: the soft-cap rule at round A, the hard cap at round A+1.
  D1's ender list, the amendment criterion that said the count starts at 0, and the plan-writer note are aligned.
- **F4 (MEDIUM), fixed differently.**
  The proposed zero-context patch-id alone was not adopted: it ignores hunk positions, so a task-side move of code reads as unchanged and would re-settle with no review (self-check, fix-proposal lens).
  D5's code identity now records the merge base, the full diff's hash and the zero-context patch-id; the subject is unchanged when the merge base and the full diff hash are equal, or when the merge base moved, every commit since the settle commit is a Merge-In merge, and the patch-id is equal.
  The diff is of the warp working tree, uncommitted tracked changes and untracked non-ignored files included; it is read through the told merge-base seam and a new `internal/fabricengine` patch reader, since `Fabric.Diff` returns paths only.
  The `_lyx` restriction is replaced by the statement that the run's records live on the weft side.
  A standalone run, which records no parent branch, has no code identity and takes case 3.
  D6 defines each segment's amendment base: a document segment's settle commit, and for a code segment the settle commit and merge base, with reviewers told the settled task change against the current one, so a Merge-In's parent changes are left out.
  The Merge-In criterion is narrowed to a Merge-In whose parent changes touch no line the task changed, and a criterion covers the code segment's amendment base.
- **F5 (LOW), fixed as proposed, extended.**
  D7: `review_amend_max_bounces` joins `reviewbudget.CheckLocal`'s bounds with minimum 0 and no null, `reviewbudget.CheckResolved` refuses a resolved value below 0, it is not a guard key, and at 0 every amendment, a PR-Gate reject's rework and a rewrite after a goto to the writer included, settles with no review.
  The `CheckResolved` bound is added because `CheckLocal` checks only a local-layer document, so a negative value in `loom.yaml` or `darn.yaml` would pass it.
  A criterion covers both refusals.

## Self-check

- fix-way-forward, MEDIUM: the verify-retry criterion named a state where the record is no longer live — fixed as proposed (criterion now names the window between the failure record and the retry).
- fix-way-forward, LOW: D7's bound covered only the local layer — fixed as proposed (`CheckResolved` bound added); merged with fix-consistency 2 and fix-proposal 2.
- fix-consistency, MEDIUM: same verify-retry criterion — duplicate of fix-way-forward's, fixed.
- fix-consistency, MEDIUM: `CheckLocal` alone — duplicate, fixed.
- fix-consistency, LOW: identity counts uncommitted changes but the reviewers' range-diff is commits only — fixed as proposed (the current side is the working tree as the identity reads it); merged with fix-claims 2.
- fix-consistency, LOW: D5's not-live list missed Merge-Review's full generation — fixed as proposed.
- fix-proposal, MEDIUM: zero-context patch-id misses task-side moves — fixed with own change (full diff hash plus patch-id under a Merge-In-only condition), see F4.
- fix-proposal, MEDIUM: `CheckLocal` alone — duplicate, fixed.
- fix-proposal, LOW: untracked files excluded from the identity — fixed as proposed (untracked non-ignored files count).
- fix-claims, MEDIUM: verify-retry criterion — duplicate, fixed.
- fix-claims, MEDIUM: reviewers' view misses uncommitted changes — duplicate of fix-consistency's LOW, fixed.
- fix-claims, LOW: no told fabric seam returns a patch — fixed as proposed (names a new `internal/fabricengine` patch reader).
- fix-claims, LOW: standalone run's code identity undefined — fixed as proposed (no code identity, case 3).

## Disputed

None.

## Deferred

None.
