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
---

## Changes

- F1: `pullrequest.MergeState.MergeabilityUncomputed` reports an open state whose only cause is the uncomputed mergeability.
  `mergeRefusalReason` returns a halt naming GitHub's message and that cause, with the way forward of waiting for GitHub and running `lym loom resume`.
  A beam's halt also names `lym beam pr status <slug>`.
  The row `AMergeabilityGitHubNeverComputesHaltsNamingItAfterTheWindow` asserts no "approves" and no "lym batten cancel".
- F2: a row for a failed merge-state read after a `RefusalOther` refusal, with no second merge.
- F3: the `doc.go` sentence, the `mergeOnGitHub` doc and the `prMergeRetryWindow` doc state the retry schedule and the extra reading wait.
- F4: the beam sentence in the `mergeRefusalReason` doc and in `doc.go` names the resume-only exception.
- F5: the `RefusalOther` comment covers GitHub's "not mergeable" while mergeability is computed, which a retry can clear.
- F6: `_lym/landing/summary.md` names the beam verbs `lym beam pr status` and `lym beam ready` in the Merge danger blast radius.
  The file is under `_lym`, so it was edited and not committed.
- Commits: F1 to F5 in one commit, whose message names each finding, then a second commit for the self-check changes.
  The findings overlap in the same files, so they could not be split per finding.

- Gate repair: six comment lines the gate flagged as wrapped breaks in `doc.go` and `finalize_pr.go` were rewritten as one sentence per line.
  No wording changed beyond the split.

## Self-check

- fix-code-correctness, LOW: the `mergeOnGitHub` doc line past 120 characters. Fixed.
- fix-code-correctness, LOW: no test row for a beam's uncomputed-mergeability halt. Fixed with a row.
- fix-consistency, LOW: the same over-long line. Merged with the one above. Fixed.
- fix-consistency, LOW: `_lym/landing/summary.md` sections stale against the code. Fixed.
- fix-proposal, LOW: "about two minutes" understates the worst case. Fixed, the docs now say about two and a half minutes.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
