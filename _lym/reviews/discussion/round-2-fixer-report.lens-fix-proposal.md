```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out as proposed, and find concrete cases where those proposals are wrong or break something else in the record.

1. **MEDIUM**: D3's new "records config's values at its next drift row or check" and D6's new "the first drift check that finds a run past Drift-Expect with no seal commit record `unsealed` in `scope.json`" (proposals F3 and F15).
   These make `lym drift check` and the drift gate entry write `scope.json`, which is a tracked record.
   Evidence: D8 still says the entry "writes no tracked record", and its Bound says "`check` and the entry write only their scratch capture directory".
   The D8 acceptance criterion says "neither writes a tracked record".
   Case: a writer at Darn-Write runs `lym drift check` on an in-flight run that has no seal. The check writes `unsealed` into `scope.json`, which D8 forbids.
   Either the check stays read-only, so the mark waits for Drift-Verify and a forge before that row still passes, or D8's bound and criterion are now false.

2. **MEDIUM**: D7's new baseline-digest paragraph and its rationale, "the digest turns such an edit into a stop instead of a pass" (proposal F1).
   The digest is recorded in `scope.json` under `_lym/drift/`, which the writer can edit as easily as the baseline.
   Evidence: support-log Turn 2 says "no guard keeps an agent from editing `_lym/drift/`".
   D6 rejected seal fields in `scope.json` for this very reason (support-log, Rejected alternatives: "an edit of file and record together could fake it").
   Case: the writer edits the baseline file and then rewrites `scope.json`'s digest to the new tree's digest. The entry passes, and so does Drift-Verify.
   The stated bound ("a stop instead of a pass") does not hold. At most the edit shows in the weft diff of `scope.json`, and D7/D8's Bound does not say so.

3. **MEDIUM**: D9's new guard way forward, "the parent reverting them in the run's worktree, for example with `git checkout <head> -- <path>`, then `lym loom resume`", together with "A resumed row compares against the recorded snapshot … until the trees match it" (proposal F8).
   Case: the judge makes a commit on the weft (or the warp), so HEAD moves, which D9 counts as a Stuck.
   `git checkout <head> -- <path>` restores file content but does not move HEAD back. The resumed row compares HEAD against the recorded snapshot and stays Stuck on every resume.
   The named way forward cannot clear a moved-HEAD Stuck. That needs a reset of the branch, which D9 does not name.

4. **MEDIUM**: D4's new "Drift-Baseline and Drift-Verify, on entry and before their own work, end through `runproc.EndRole` any chair of the removed Drift-Baseline or Drift-Verify agent" (proposal F11).
   `EndRole` ends by role name (`internal/runproc/end.go:69`, `EndRole(dir, owner, role string, callerPID int)`), and the old verify seat's role is `drift-verify` (`internal/driftengine/seats.go:27`).
   D9 does not name the judge's role. If the judge seat keeps the Drift-Verify seat's role, then Drift-Verify's entry sweep ends a live interrupted judge before D9's probe-and-resume rule can resume it.
   The proposal needs the judge's role kept distinct from the removed agents' roles, or the sweep limited to sessions spawned by the old stencil. The record states neither.

5. **LOW**: D7's "A check that finds the lock held waits for it rather than refusing", with the slot taken first (proposal F10).
   A check waiting on the lock holds a gate slot for the whole of another check's capture, so with one free slot every other gate test in the hub waits behind both captures.
   This does not deadlock, but D7's rationale says checks "serialise without deadlock" and does not state the bound that the slot pool is held idle while a check waits.

No further tool calls made.
