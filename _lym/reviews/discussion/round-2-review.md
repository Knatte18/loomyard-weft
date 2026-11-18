---
verdict: BLOCKING
findings:
  - id: F12
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D4
    summary: D4's fix-here count, added for F7, clears when an episode of PR-Review's segment begins, but the PR-Review row has no segment in any recipe and its Call cannot tell a new episode from a re-step, so the count either carries across a send-back and the next small set sends the run back again until the reroute cap halts it, or clears on every re-step and the per-episode cap is lost.
    fix: In D4, replace "The PR-Review row clears that count when an episode of its segment begins." with a rule that the row clears the fix-here count together with the send-back mark, parent_review_asked and the session id when it reroutes on a clean tree, and that nothing else clears it, so a re-step, a crash-resume and the resume after the dirty-tree Stuck keep it. In D4's Bound, change "one episode fixes at most pr_review_small_max findings in place" to say at most pr_review_small_max between two send-backs, or before the first. Replace the criterion "D4 - a new PR-Review episode starts with a fix-here count of zero." with one stating that a PR-Review row re-entered after a send-back starts with a fix-here count of zero, and that a re-step of the row keeps the count.
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: D4 now bounds sub-threshold sets with a fix-here count and states skipping or shaping the verb as an accepted risk with its limits; the count's undefined reset point is a defect of that fix, raised as F12.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
    answer: The fixed-differently halt on a dirty tree, with commit or discard then lym loom resume as its way forward, settles the stall the proposed wait would cause, and a criterion covers it.
  - id: F10
    status: resolved
  - id: F11
    status: resolved
---

### [BLOCKING] The fix-here count has no defined reset point

**Location:** `_lym/discussion/decision-record.md#D4`

**Issue:** The F7 fix added a fix-here count to the pull-request record that "the PR-Review row clears when an episode of its segment begins".
The PR-Review row declares no `segment:` in the tapestry, darn or sampler recipe (`contracts/recipes/tapestry-recipe.yaml:523-525`).
A row without a segment ends its shed episode only at its own Done (`internal/shedengine/run.go`, `episodeStuckCount`).
A reroute ends only the episodes of rows sharing the target's segment (`internal/shedengine/doc.go:112`), which here is Code-Review.
`PRReview.Call` is re-entered on every re-step and resume and sees no episode boundary (`internal/landingshed/prreview.go:67-86`).
The clean-tree send-back clears the mark, `parent_review_asked` and the session id, but not the count.
If the implementer follows shed's episode rule, the count carries across the send-back: after Code-Review has re-reviewed the whole range, the first sub-threshold set sends the run back again, until `rerouteCap` halts it.
If they clear it on each `Call`, a transient push or GitHub failure, a crash-resume or the resume after D4's own dirty-tree Stuck zeroes it, and D4's Bound loses the cap the F7 fix relies on.
The criterion "a new PR-Review episode starts with a fix-here count of zero" inherits the same ambiguity.
The walkthrough and existing-paths lenses found it too, and the interactions lens at MEDIUM.

## Rejected

- Raised at MEDIUM or LOW this round, and so outside the incremental round's rule that a new finding is BLOCKING, are the observations below.
  They are not false positives; the parent may want them before approval.
  - interactions 2, existing-paths 6 and bound-breaker F-B: D4 says neither whether the send-back mark or a continue flag set by `lym loom pr-continue` wins in the wake, nor whether the reroute clears `Continue`. On a clean gated tree `judge` ends the review Done on the continue flag, so a run can land with a send-back not taken or without the re-ask.
  - walkthrough 2 and existing-paths 2: D3's `uncaused` Stuck goes to the parent with "a commit carrying the trailer … then `lym loom resume`" as its way forward, but names no session that makes that commit. Every existing drift Stuck routes a code fix through `lym loom goto --to <writer row>`.
  - walkthrough 3, existing-paths 4, bound-breaker F-G and interactions 3: D5 lets a task's `run_settings` set `pr_review_small_max` but states no config category. Under `PATTERN-config-layers` a key whose large value hollows a review gate is Guard and closed to the local layer; as None it lets any task turn D4 off, an unstated bound.
  - bound-breaker F-C: the send-back file sits under `.lym/loom`, which a fresh machine starts without, while the send-back mark sits in the tracked pull-request record. A handover between the send-back and the re-entered Code-Bouncer loses the review input.
  - bound-breaker F-D: at the reroute cap the row has already cleared the mark before shed halts it, so the resume the cap's way forward names never sends back those findings.
  - existing-paths 3: during a send-back the pull request stays open and ready on GitHub, so it can be merged while the run is back in Code-Review.
  - bound-breaker F-E: D2's trailer commit "with no change if need be" needs `git commit --allow-empty`; the record pins neither the flag nor that the commit paths accept an empty commit.
  - bound-breaker F-H and existing-paths 5: with Tier 2 off, `friction.NotePath` returns an empty path and D6 records no miss.
  - existing-paths 6 and bound-breaker F-F, LOW: the mark forces no status poll, so the wake acts on it up to a minute late. A trailer committed after Drift-Recheck never reaches the regression check section D2's Bound cites.
