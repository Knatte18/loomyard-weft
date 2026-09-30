---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (Decisions, new gated shape; Constraints; Acceptance criteria, destroy.go unit tests)
    summary: Reusing the managed-branch ownership kind refuses every step-12 deletion, because the new local weft branch of the same name is checked out at the new weft worktree by then
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decisions, weft refusal remedies)
    summary: The refusal remedy lyx fabric remove is unusable when the pair is already gone from this hub, which covers every leftover from before archive tagging landed on 2026-09-30
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decisions, archive tag is the orphan proof; Open risks, third bullet)
    summary: The ancestor check needs the archive tag targets locally, but the record only fetches the remote branch tip, so a re-add on another hub refuses an archived leftover
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Goal)
    summary: The Goal promises the re-add succeeds deterministically, but the warp and unarchived-weft decisions make some plain-Remove re-adds refuse
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (Scope Out; Acceptance criteria, lease race)
    summary: The rollback's bare-slug warp-branch residue after a step-12 failure is neither in nor out of scope, and the lease-race criterion says nothing about it
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions, pre-flight weft outcomes; Auto-mode assumptions, adopt path)
    summary: The adopt path keeps an unprobed push that is rejected mid-Add when origin is ahead of the surviving local weft branch, contradicting the Goal
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria, lease race)
    summary: The lease-race criterion names no seam that moves the remote branch between probe and deletion, so a timing-independent test is left for the plan writer to invent
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions, warp side rationale; Auto-mode assumptions, last bullet)
    summary: The warp rationale's step-11 failure narrative and the why-in-scope assumption are deliberation that belongs in support-log.md
  - id: F9
    severity: NIT
    location: _lyx/discussion/decision-record.md (Open risks, first bullet)
    summary: The first Open risks bullet says at an archived tip where the proof accepts any tip reachable from an archive tag
---

### [BLOCKING] Managed-branch ownership refuses the step-12 deletion

**Location:** `decision-record.md`, Decisions ("The new gated shape lives in `destroy.go`…"), Constraints (first bullet), Acceptance criteria (`destroy.go` unit tests), Notes for the plan writer (second bullet).

**Issue:** The record declares the new deletion's ownership as "the weft-suffix managed-branch kind the existing remote deletion uses", i.e. `ownedManagedBranch`.
Its predicate, `resolveManagedBranch` (`internal/fabricengine/destroy.go`), lists the weft repo's local branches and refuses when a branch of that name "is checked out at" a worktree.
The deletion is decided to run at step 12, after `createWeftWorktree` has created the new local `<slug>-weft` and checked it out at the new weft worktree.
The remote ref and the new local branch share the name, so the ownership check refuses every replacement, and the deterministic-divergence criterion can never pass.
Replacing the dirtiness kind does not help: the checked-out test lives in the ownership predicate too.
The same kind also accepts any branch under a non-empty `branch_prefix`, so the criterion "a branch without the weft suffix refused on ownership" does not hold for it.
Separately, Constraints calls the deletion "a new request shape", while the Notes extend the existing `remoteBranchRequest`; the plan writer gets two answers.

**Fix:** Decide a new closed-enum branch-ownership kind for this deletion: the branch is exactly the pair's own weft branch (`WeftBranchName(warpBranch)`), weft-suffixed, and not the repo's primary weft branch, with no local-checkout test, since the same-named local branch is the replacement.
State that the deletion extends `remoteBranchRequest` with that ownership kind, the new dirtiness kind and the lease SHA, and make Constraints say the same.
Keep the unit criterion "a branch other than the pair's weft branch refused on ownership".

### [MEDIUM] The weft refusal's remedy fails for a pair already removed here

**Location:** `decision-record.md`, Decisions ("The weft refusal names the branch, the remote tip, and the remedies").

**Issue:** The only in-product remedy is `lyx fabric remove <slug>` on the hub that owns the pair.
When the pair was removed on this very hub, `Remove` fails with "worktree … not found", so the remedy does nothing.
That is the common case, not a corner:
archive tagging landed in `676d9ec03` on 2026-09-30, so every leftover remote weft branch a plain Remove left before that commit has no `archive/<slug>/*` tag and is refused.
The same holds when Remove ran with `ArchiveSkippedReason` (no origin then), and when the local weft branch was behind or diverged from origin at Remove time: `archiveWeftTip` tags the local tip, the remote tip is not reachable from it, and Add refuses.
Refusing in those cases is correct, since the proof holds in the safe direction and never deletes unarchived content, but the operator is left with "pick a different slug" as the only remedy that works.
`support-log.md` rejects "plain Remove deletes the remote branch by default" partly because it "does not help hubs whose leftovers already exist", which applies equally to the chosen design.

**Fix:** Word the remedies by case: if a pair of that slug is live on some hub, remove it there (`lyx fabric remove <slug>` archives it); if no hub holds the pair, delete the remote branch by hand once its content is known to be disposable (`git push origin --delete <slug>-weft`), or pick a different slug.
Add an acceptance criterion that the refusal names the hand-deletion remedy.
Correct the rejected-alternative rationale in `support-log.md` so it no longer claims an advantage the chosen design lacks.

### [MEDIUM] Cross-hub reachability lacks the archive tag's objects

**Location:** `decision-record.md`, Decisions ("The archive tag is the orphan proof"), Open risks (third bullet), Notes for the plan writer (first bullet).

**Issue:** The proof accepts a remote tip that equals or is an ancestor of an archive tag's target.
`ls-remote --tags` gives the target SHA, and `archiveWeftTip` creates lightweight tags, so equality is a plain SHA comparison.
The ancestor case needs `merge-base --is-ancestor <remote tip> <tag target>` with both commits local.
On the hub that ran Remove the tag target is local, because `archiveWeftTip` created it there.
On any other hub, fetching the remote branch tip, the only fetch the record names, does not bring the tag target.
The check then fails, and Add refuses an archived leftover, for example one whose local tip was ahead of origin when another hub removed it.

**Fix:** State in Decisions that the probe compares SHAs first, then fetches the remote branch tip and the matching `refs/tags/archive/<slug>/*` targets into `FETCH_HEAD` without creating local refs, and that a commit still missing afterwards counts as unreachable, so Add refuses.
Update the Open risks bullet to name both fetches.

### [MEDIUM] The Goal contradicts the refusal outcomes

**Location:** `decision-record.md`, Goal.

**Issue:** The Goal says `lyx fabric add <slug>` after a plain Remove "succeeds deterministically".
The warp decision and its criterion make a squash-landed re-add refuse until someone deletes the remote warp branch by hand.
The weft decision makes an unarchived leftover refuse, which with F2 covers every pre-2026-09-30 leftover.
Scope and the acceptance criteria describe the refusals; the Goal does not.

**Fix:** Make the Goal say that a re-add succeeds when the leftover remote weft branch is covered by an archive tag and any remote warp branch is fast-forwardable, and that every other leftover produces a pre-flight refusal with a working remedy.

### [LOW] Rollback's warp-branch residue is unscoped

**Location:** `decision-record.md`, Scope (Out), Acceptance criteria (lease race).

**Issue:** The warp rationale cites the rollback's inability to delete a bare-slug warp branch under the default empty prefix.
The new step-12 failures (lease mismatch, branch protection) still reach that rollback after step 11 has pushed the warp branch.
Such a failure leaves the local warp branch and its remote copy, and the next retry hits `ErrBranchExists`.
The record neither scopes that residue out nor tells the lease-race criterion what state to expect.

**Fix:** Add an Out bullet stating that the rollback's existing warp-branch residue is unchanged by this task, and make the lease-race criterion check only what this task owns: the weft side is rolled back and origin keeps the moved tip.

### [LOW] Adopt path keeps a mid-Add push rejection

**Location:** `decision-record.md`, Decisions (pre-flight weft outcomes, second sub-bullet), Auto-mode assumptions (fourth bullet).

**Issue:** A plain Remove normally deletes the local weft branch: `removeWeftWorktree` deletes it through the gate once the worktree is gone.
The branch survives when that deletion fails and Remove tolerates it because the weft worktree directory is already gone, for example a stale worktree registration that makes the gate see the branch as checked out.
Re-add then adopts it with no probe.
If origin holds commits the local branch lacks, step 12's push is rejected after the whole pair is built, which is the exact failure the Goal says never happens.
The assumption that the adopted branch "already matches or fast-forwards" its remote copy is unprobed.

**Fix:** Give the adopt path the warp side's check: `ls-remote` for the weft branch, then `merge-base --is-ancestor <remote tip> <local weft tip>`, with a missing object counting as not an ancestor, and a pre-flight refusal naming the branch.
Replace the Auto-mode assumption with that decision, and add an acceptance criterion for it.

### [LOW] Lease-race criterion has no deterministic seam

**Location:** `decision-record.md`, Acceptance criteria (lease race).

**Issue:** No part of `Add` runs caller code between the pre-flight probe and step 12.
A test can only move the remote branch in that window through a test seam, or through a concurrent pusher, which is timing-dependent and contradicts the Goal.

**Fix:** Name the seam in the criterion: a test-only hook that `Add` calls after pre-flight and before the deletion, through which the test pushes onto origin's weft branch.
Also add a `destroy.go` unit criterion that drives the leased deletion with a stale lease SHA and expects the deletion to fail and the remote ref to be unchanged.

### [LOW] Deliberation narrative in the decision record

**Location:** `decision-record.md`, Decisions (warp side, rationale), Auto-mode assumptions (last bullet).

**Issue:** The warp rationale retells today's step-11 failure and the rollback's residue.
`support-log.md` Interview 5 and Q5 already hold that account.
The last Auto-mode assumption explains why the warp pre-flight is in scope, which is a scoping argument, not an assumption the plan relies on.
Completeness check: the only requirement in either passage is "fabric performs no remote warp deletion", which the warp decision already states.

**Fix:** Cut the warp rationale to the policy sentence plus one clause saying today's failure happens after the pair is built.
Delete the last Auto-mode assumption; Q5 in the support log already records it.

### [NIT] "At an archived tip" understates the proof

**Location:** `decision-record.md`, Open risks (first bullet).

**Issue:** The proof accepts any remote tip equal to or an ancestor of an archive target, and the bullet says "at an archived tip".
The case the focus directive raises, a live pair elsewhere whose remote tip is only the parent's weft commit, does not arise from fabric's own Add: Add always commits the origin record on top of the parent's weft commit and pushes it at step 12.
The case that does arise is the same pair checked out on two hubs, with the other hub's pushed tip inside the archived history.
The bullet and the one-live-hub assumption describe that case correctly once the wording matches the proof.

**Fix:** Reword to "a tip equal to or an ancestor of an archive tag's target".
