# Fixer-report — code round 1

F1 is `decision`-class and went to the parent; it is left out here.

## Changes

All in `docs/research/creel-hook-delivery-2026-10-25/report.md`.

- F2, fixed as proposed (commit `37b048414`): the Method bullet says which figures come from uncommitted runs.
  The self-check below narrowed the wording further.
- F3, fixed as proposed (commit `409c168cd`): median 4.1 s changed to 3.9 s; checked against `claude-mid-turn-30.tsv` (15th 3.748 s, 16th 4.118 s).
- F4, fixed as proposed (commit `0ccd04cd1`): the Recommendation now says about 0.6 ms over a bare process start.
  The twin sentence in the Hook cost bullets ("adds under 1 ms") got the same change.
- F5, fixed as proposed (commit `4ad79aec7`): the continuation line is indented under its list item.
- F6, fixed as proposed (commit `4ad79aec7`, F5 and F6 share one commit): lines 61, 83, 100, 101, 115, 122, 126 and 130 are broken at the clause boundary.
- Self-check follow-up (last commit of the round): the Method sentence names the hook's 3.7 s and the 25 ms claim latency as uncommitted, and the re-derivable set is the 30-message figures plus the SendMessage 31.7 s and 1.9 s.
  Every remaining line in prose bullets and paragraphs with a `;` between two clauses is also broken there.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the F2 sentence overstated the re-derivable figures. Fixed as proposed.
- fix-proposal, LOW: same finding as fix-consistency, merged. Fixed as proposed.
- fix-affected-set, LOW: F6's set is larger than the named lines. The `;` lines are fixed; the `, and` / `, but` joins on lines the review did not name are deferred (see below).
- F3 and F4 sets: covered, per the same lens.

## Disputed

None.

## Deferred

- F1: `decision` class, left to the parent.
- F6 remainder: lines outside the review's list whose two clauses join with `, and`, `, but` or `, so`, in prose bullets and paragraphs.
  The review's `fix` named eight lines; the broader set is a style sweep of the whole report that the reviewer did not ask for, and a regex cannot tell a compound predicate (exempt) from two clauses, so each needs a reading the fixer left to a next round.
