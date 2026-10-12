---
round: 2
focus:
  - "Parent ref consistency in `_lyx/discussion/decision-record.md` D1 and D9: the ref Merge-In merges (today `mergeresolve.Resolve` merges a local branch name via `fabricengine.MergeIn`, with no fetch), the ref recorded as the merged parent tip, and the ref D9's moved-parent check reads after a fetch of the upstream. Settle whether a hand push to the parent's upstream leads to a rerouted Merge-In that merges the pushed commits and a passing next check, and what happens when the local parent is ahead of or diverged from its upstream."
  - "Lock release on rejection in D5 and D9: PR-Gate ensures the parent lock, and a matched rejection takes its Stuck exit to the rework row, which walks the main line again. Settle whether any Decision releases the run's lock on that edge, so other runs do not wait at Merge-In through the whole rework, and whether Merge-In reacquires afterwards."
  - "Finalize's parent-side merge retry in D9: `Finalize.Call` re-runs the catch-up merge-in and retries the parent-side merge when the parent moved after the first catch-up. Settle whether a parent code-side commit arriving between Finalize's check and that retry can land without a merge review, and what bounds the retry."
  - "Lock record ownership after a PR-Gate resume in D5 and D8: batten holds a record for the child, and D5's loop exit hook releases the run's lock. Settle who releases it, by run or by owner process, if the resumed child stops before taking the record back, and what bounds how long a live batten keeps it."
---

Focus questions only; the reviewer still reviews the whole pair against the rubric.
