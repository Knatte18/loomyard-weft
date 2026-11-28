```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every bound, behaviour and code fact that the changed text asserts against the code at HEAD.

**MEDIUM: the batch gate's rerun claim contradicts the code (D7, and the D4 sentence that depends on it).**
- The changed text in D7 says: "The batch gate runs `verifyGate`'s check and rerun with no failure recorder and no flaky callback". The fix took this from the reviewer's F14 proposal.
- The code says otherwise. The field doc at `internal/landingshed/verifygate.go:47-49` reads: "onFlakyPass, when set, makes check run a failure of test runs once more on the same head before judging it. It gets each failure that rerun did not repeat; nil runs no rerun."
- So with no flaky callback there is no rerun at all. That contradicts D7's own opening clause "after landingshed's rerun of a test-run failure".
- It also breaks D4's "Batch-gate failures a rerun did not repeat … the leader logs them at Warn". Those failures only reach anything through `onFlakyPass`, so with a nil callback nothing exists to log.
- What the text needs to say: the batch gate runs with a flaky callback of its own, one that logs at Warn with the chain commit and does not touch the leader's flaky list. Only the failure recorder is nil.

**LOW: the rerun wording in D7's Bound misstates which failures get rerun.**
- The changed text reads: "a tier timeout or build failure, which the rerun does not repeat, blames entry 1 or the prefix where it first shows".
- `rerunTestFailure` (`internal/landingshed/verifygate.go:306-318`) returns a timeout, a build failure or an unjudged run as it is. These are never rerun.
- "Does not repeat" is the phrase the code uses for a failure that did not recur on rerun. Here it reads as if those failures were rerun and then passed.
- Suggested wording: "which the rerun does not run again".

**Claims that hold, so no finding:**
- D5's darn scope bound. `darnTreeScope` diffs `taskMergeBase` (`MergeBase(record.MergedTip)`, `internal/loomcli/subjects.go:143`) against HEAD, so a HEAD at the chain commit covers every member's trees.
- D1's `needsGitHubOnDisk` runs before the lock today (`finalize.go:384`, `:432`).
- D9's retired-key layer behaviour, per `configengine/doc.go:71-75` and `landingshed/config.go:184`.
- `Deps.RemoveLeftovers` is the merge-in's seam (`deps.go:232-239`).
- D6's `oldestHeldRecords` fallback (`landingdeps.go:982-993`).
- D9's count of two extra gates for four entries.

Tool calls I made, because my context did not have this code:
1. I grepped `taskMergeBase` and `verifyGate`'s fields, to check D5's darn scope claim and D7's recorder and callback claim.
2. I read the rest of `taskMergeBase` (`subjects.go:142-160`) and the `onFlakyPass` field doc (`verifygate.go:44-50`), to confirm which merge base the darn diff uses and what a nil callback does.
