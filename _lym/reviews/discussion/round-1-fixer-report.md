---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: The proposed wait on a dirty tree stalls the row, since the verb has told the agent to end its turn and nothing prompts it to commit or discard; the wake instead stops the session, re-reads the tree and on a dirty one halts Stuck for the parent with commit or discard then lym loom resume as the way forward.
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md`, with a round entry and ledger lines in `_lym/discussion/support-log.md`.

- F1: D3 keys a stored ruling on the file's diff hash together with the range's `Loosened-Check:` trailer lines, so a trailer-only commit re-rules the file; D3's Bound names the extra judge session.
  New D3 criterion: a trailer-only commit after an `uncaused` Stuck, then `lym loom resume`, re-spawns the judge, rules the check `caused` and lets Drift-Verify finish Done.
  The reuse criterion now names unchanged diffs and trailers.
- F2: D4 stores the send-back file as `pr-findings-<n>.md` in the loom scratch directory beside the Parent-Align fixer report, outside the archived `_lym/reviews/code/` run directory; D6 names that file.
  The re-entry criterion requires the reviewers to read it.
- F3: D4 answers fix-here when the target perch approves without review, as with a Code-Review budget of 0 stamping `zero_rounds` on Code-Bouncer, decided from built rows and settings; the fix-here criterion carries the case.
- F4: D4 states the new mechanism: a send-back file newer than the settle forces a full generation even on an unchanged subject, the `ReviewInput` seam returns every input written after the settle, the generation record keeps each path, and a review-input note outside the amendment note names them.
  The Rationale lists these as new.
  D6's rubric sentence reads inputs from the review-input note too.
- F5: the reroute clears `parent_review_asked`; the re-entry criterion requires a new review request with `pr_review_ask_parent` on.
- F6: the reroute clears the recorded session id, so the re-entered PR-Review starts a new session; the criterion says so.
- F7: a fix-here count in the pull-request record, cleared when a PR-Review episode begins, bounds successive sub-threshold sets; skipping or shaping the verb is an accepted risk with its limits stated in D4's Bound.
  D2's Bound sentence matches, scoped to where the edge can fire. Two new D4 criteria cover the count.
- F8: D2's Bound states that a trailer's cause is the committer's attestation, not re-measured, and that its bound is the regression check section in the pull request body and, with `pr_review_ask_parent` on, the parent's review.
- F9 (fixed differently): the wake that meets the mark stops the session and re-reads the tree; on a clean tree it reroutes, and on a dirty one it halts Stuck for the parent with commit or discard then `lym loom resume`.
  The reviewer's "keeps waiting on a dirty tree" was not made, because the agent is idle after the verb's answer and the row would wait with nobody to act.
  A criterion covers the dirty case.
- F10: new D2 criterion that a Go-written commit carries the trailer lines the role's handoff gives.
- F11: the Auto-mode bullet is replaced by a pointer to D4's rule and the question ledger.

## Self-check

- fix-claims MEDIUM, fix-here cap fails in the fallback cases (merged with fix-consistency MEDIUM): fixed; D4's Bound and D2's Bound scope the cap to runs where the edge can fire and name the uncapped fallback.
- fix-claims MEDIUM, parent review before merge not guaranteed: fixed; the accepted-risk limit now says the row waits for no review and the parent is asked only with `pr_review_ask_parent` on.
- fix-claims LOW, D2's parent's review needs `pr_review_ask_parent`: fixed; D2's Bound says so.
- fix-consistency MEDIUM, stop contradicts "session up" on a dirty tree (merged with fix-way-forward MEDIUM and fix-proposal MEDIUM, idle agent stalls the row): fixed; a dirty tree after the stop halts Stuck for the parent with its way forward, and the criterion matches.
- fix-consistency LOW, D6 rubric criterion stale: fixed; the criterion requires reading inputs the review-input note names.
- fix-consistency LOW, which note names the fixer report on an amendment: fixed; the amendment note keeps naming it, the review-input note serves a full generation, and D6's rubric reads both.
- fix-way-forward LOW, fix-here count not scoped to the episode (merged with fix-proposal LOW): fixed; the row clears the count when a PR-Review episode begins, with a criterion.

## Disputed

None.

## Deferred

None.

## Rulings

None.
