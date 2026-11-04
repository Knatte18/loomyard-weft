---
round: 1
focus:
  - "Whether every consumer of beam.yaml's `pull_request` value set learns `merge`: grep the repo for `required` beside `pull_request` (other validators, `lym config beam` set paths, docs and specs the repository's rules name for a config-key change) and settle which still enumerate only on, off and required."
  - "`internal/landingshed/finalize_pr.go` `openAtOnce` and `mergeOnGitHub`: settle whether a re-entry after a crash between `Create`, `WriteRecord` and `Merge` merges the recorded pull request exactly once, and whether the record the open path writes matches what the resume path (`StateOpen`, `StateClosed`, merged) expects."
  - "`pullRequestLanding` in `finalize_pr.go`: settle whether a MergeAtOnce yes with an existing record but a stale `GatedHead`, or a merged or closed recorded pull request, takes the intended path, and whether the no-record branch skips any check the record-bearing branch performs."
  - "`pushParent` in `finalize.go`: settle whether `protectedBranchRefusal` can match a refusal that is not the parent push's protection (marker text broad enough to misfire), whether it overrides a transient or hold-back classification it should not, and whether the Stuck reason still reaches `stuckOrCancelled` with the same error class as the other push failures."
  - "The decision record's Design bullets against the diff: pull request always opened, merge always a squash, parent lock, merged-tree gate and board done mark unchanged, a refused protected push naming the setting; settle each against the code, including whether the design's 'pulls the parent' step after the remote merge is the existing fetch and fast-forward and whether the gate of the merged tree still runs on the MergeAtOnce path."
  - "`ResolvePullRequest` and `PRReview.decide`: settle whether MergeAtOnce is reachable only from beam.yaml `merge`, whether the empty-branch and missing-origin-fact paths for a merge parent halt or fall to the no path as the doc comment says, and whether PR-Review and Finalize still cannot disagree."
  - "Test coverage of new behavior: MergeAtOnce open path with no record, with an existing open pull request, and with a recorded closed one; the method-refusal reason under atOnce; the protected-branch Stuck reason; `dobbyengine` accepting `merge`. Name any added behavior with no test, and any new test that repeats an existing case's coverage."
  - "Commit message and doc claims: the commit body states PR-Review passes at once and a refused protected push names the setting; check the `doc.go` paragraphs and the comment on `pushParent` against the code, including the 'Bound' line about a run marked done while its work is on the local parent only."
---

Seed focus for round 1, set from the decision record's `## Goal` and the committed diff over the review range.
Entries name where to look and the question to settle; none states a verdict.
