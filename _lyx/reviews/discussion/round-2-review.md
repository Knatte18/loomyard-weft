---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D9 (moved-parent check and Finalize retry), D1
    summary: D9's moved-parent check reads only the parent's upstream tip, but Finalize's catch-up merge-in and parent-side merge use the local parent branch, so a local-only parent commit passes every check and lands through the catch-up unreviewed
    fix: In D9, define the moved-parent check over both refs - the parent has moved when either the local parent branch tip or its upstream tip after a fetch differs from the merge record's merged tip (the local tip alone for a parent with no upstream), and the parent does not already contain the task's work; state that a local parent ahead of or diverged from its upstream at Publish, PR-Gate or Finalize reroutes to Merge-In, where D1's Stuck applies. Apply the same two-ref read to the condition on Finalize's retry after merge-in-required. Amend the acceptance criterion on a commit arriving between Finalize's check and its parent-side merge to cover a local commit on the parent branch as well as a push to its upstream, and replace the sentence saying Finalize's catch-up merge-in brings no new code with one that holds under the two-ref check.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 (fast-forward of the local parent branch)
    summary: D1's fast-forward of the local parent moves the parent pair's checked-out worktree, but names neither that pair, nor its place relative to the lock acquire, nor the Stuck for a dirty, mid-merge or missing parent pair
    fix: "In D1, state that the fast-forward pulls the parent pair's own worktree through fabricengine's pull, the pair Finalize already opens, and that it runs after Merge-In holds the parent lock (D5). Add Stuck exits with ways forward for a dirty parent pair, a parent pair with a merge in progress, and no live parent pair, each naming the step in the parent pair and then lyx loom resume. Add the bound: the fast-forward only moves the parent pair to its upstream tip when the move is a fast-forward and never touches local commits. Add matching acceptance criteria."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1 (local parent ahead of its upstream)
    summary: under FABRIC_SKIP_PUSH, Finalize never pushes the parent, so the local parent runs ahead of its upstream after the first landing and D1 makes every later Merge-In on that parent Stuck
    fix: In D1, state that under the told push skip (Deps.PushSkipped), Merge-In merges and records the local parent tip without the fetch and fast-forward, as for a parent with no upstream, and D9 compares the local tip; add an acceptance criterion for two successive landings on one parent with the push skipped.
---

### [BLOCKING] The moved-parent check reads a different ref than the merge that follows it

**Location:** `_lyx/discussion/decision-record.md` D9, with D1.

**Issue:** D9 reads "the code side's parent tip as D1's Merge-In reads it, from its upstream after a fetch", and concludes that "Finalize's catch-up merge-in stays, after this check, so it brings no new code".
That conclusion does not hold.
`Finalize.mergeInStep` (`internal/landingshed/finalize.go`) calls `fz.resolver.Resolve(ctx, fz.deps.ParentBranch)`, and `mergeresolve.Resolve` merges that local branch name through `fabricengine.MergeIn` with no fetch.
The parent-side `Merge` in Finalize's steps 4 and 5 also runs in the parent pair's local worktree.
A commit made on the local parent branch after Merge-In, for example the orch committing a fix in the prime before pushing it, leaves the upstream tip equal to the recorded tip.
D9's check then passes at Publish, PR-Gate and Finalize.
Finalize's parent-side merge reports merge-in-required, the retry condition ("the parent's code side still equals the merge record's merged tip", read the same way) passes again, and the catch-up merge-in brings the local commit into the task and lands it with no merge review.
That is the exact path the Goal and the rejected alternative "Letting Finalize's catch-up merge bring new parent code" exist to close.
D1 already treats a local parent ahead of its upstream as a state that must not be merged, so the record contradicts itself: the ahead state is refused at Merge-In and goes unseen at every later row.

Focus question 1 verdict: a hand push to the upstream is handled.
D9 sees the upstream tip differ, the rerouted Merge-In fast-forwards the local parent and merges and records the pushed tip, and the next check passes, subject to F2.
A local parent ahead of or diverged from its upstream is Stuck at Merge-In, as D1 states, but only at Merge-In; past it, this finding applies.

Focus question 3 verdict: the lock keeps another run from landing between Finalize's check and the retry, and the retry is bounded by the existing single retry plus D9's three-reroute cap.
A commit from outside lyx is not bounded: a push to the upstream between the check and the parent-side merge leaves the later push of the parent rejected as non-fast-forward, which is the existing "push it by hand" Stuck and lands nothing unreviewed on the upstream; a local commit lands, as above.

### [MEDIUM] D1's fast-forward moves another pair's worktree with no stated actor, ordering or refusal

**Location:** `_lyx/discussion/decision-record.md` D1.

**Issue:** D1 says Merge-In "fast-forwards the local parent branch to the upstream tip through `fabricengine`".
The local parent branch is checked out in the parent pair's own worktree, the one Finalize opens through `Deps.OpenParentFabric`, so a fast-forward has to move that worktree.
In `fabricengine` that is `Fabric.Pull` on the parent pair (`internal/fabricengine/pull.go`), which also pulls the parent pair's records side, refuses with `ErrWarpDirty` on a dirty worktree, and refuses with `ErrMergeInProgress` while a fabric merge record exists.
D1 names Stuck exits only for a local parent ahead of or diverged from its upstream.
A dirty prime, which is the parent pair for `main`, or a parent pair with a merge in progress makes Merge-In fail with no way forward, against the record's own `PATTERN-refusal-way-forward` constraint.
D1 also leaves open whether the fast-forward runs before or after D5's lock acquire.
Before the lock, it can race another run's Finalize, which merges into the same parent worktree under the lock.
By the rubric's attack-surface question, a row that now mutates another pair's worktree needs a stated bound, and D1 states none.

### [LOW] A push-skipped hub sticks every Merge-In after its first landing

**Location:** `_lyx/discussion/decision-record.md` D1.

**Issue:** `FABRIC_SKIP_PUSH=1` sets `SyncOptions.SkipPush` (`internal/fabricengine/fabric.go`), which reaches Finalize as `Deps.PushSkipped` and skips the parent push in `pushParent`.
After one landing on such a hub, the local parent is ahead of its upstream.
Under D1, every later Merge-In on that parent is then Stuck with a way forward that says to push, and that push is the one the setting skips.
D1 carves out a parent with no upstream but not a hub that never pushes.

## Focus answers that found no defect

- Lock release on rejection (focus 2): D5 releases the run's lock on any edge out of the Merge-In to Finalize window and names PR-Gate's rejection Stuck to PR-Rework and to Darn-Write.
  The rework comes back through Merge-In, which reacquires.
  A record batten holds at that point is first taken back by PR-Gate's re-call ("each first ensures the lock"), so the edge release finds the loop as owner.
  No defect.
- Lock record ownership after a PR-Gate resume (focus 4): D8 has batten release a handed record once a resumed child stops without taking it back, and D5's compare-and-release leaves batten's record alone at the stopped loop's exit.
  A dead batten's record is stale by pid under D4.
  While the resumed child runs, batten's hold lasts only from the resume to PR-Gate's ensure at its re-call.
  A live but hung loop in that window falls under the record's stated open risk on hung owners.
  No defect.
