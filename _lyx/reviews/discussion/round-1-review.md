---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:127 (D9), with D1 at line 32
    summary: D9 compares the parent's upstream tip after a fetch, but Merge-In merges the local parent branch, so after a hand push the reroute re-merges the same stale tip and loops to the reroute cap, and a local parent ahead of its upstream reads as moved on every check
    fix: In D1, state which ref Merge-In merges and records as the merged parent tip, and make it the same ref D9 later reads. For example, Merge-In fetches the parent's upstream, brings the local parent branch to that tip before it merges, and records it. Also state who fast-forwards the local parent pair's branch after a hand push to the upstream, and what Merge-In does when the local parent has diverged from its upstream, for example Stuck with a way forward. In D9, compare against that same recorded ref. Extend the acceptance criterion "A hand push to the parent's upstream is seen as a moved parent" so that the rerouted Merge-In then merges the pushed commits and the next check passes.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:79 (D5), with D9 at line 134
    summary: PR-Gate ensures the lock and then routes a rejection Stuck to PR-Rework, and no Decision releases the lock on that edge, so the run holds the parent lock through the whole rework main line while every other run on the parent waits at Merge-In
    fix: In D5, add a release path. A lock-holding row whose verdict takes an edge out of the Merge-In-to-Finalize window releases the run's lock on that edge. Today that edge is PR-Gate's on_stuck to PR-Rework in loom and PR-Gate's rework row in darn. Merge-In reacquires when the rework comes back around. Add an acceptance criterion that a rejected PR-Gate releases the lock before PR-Rework runs, and the reworked run reacquires at Merge-In.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:136 (D9)
    summary: D9 says Finalize's catch-up merge-in brings no new code because it follows the check, but Finalize's existing retry, which re-merges the parent when the parent-side merge reports merge-in-required, merges whatever the parent moved to after the check and lands it with no review
    fix: In D9, add a sentence on Finalize's retry after a merge-in-required parent-side merge. The retry may run only when the parent's code side still equals the merge record's merged tip, which covers a move by run records alone. Otherwise Finalize reroutes to Merge-In as its first check does. Add an acceptance criterion that a code-side parent commit that arrives between Finalize's check and its parent-side merge reroutes and never lands.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:117 (D8), with D5 at line 82
    summary: After a resume, batten's held record has no stated bound until the child takes it back, and D5 does not say whether the loop's exit hook releases by run or by owner process, so a resumed child that stops before reaching a lock-taking row can leave the lock with a live batten indefinitely
    fix: In D5, state whether the exit hook releases a record that names the run but is owned by another process. Prefer a compare-and-release on the loop's own pid and start time, matching D8. In D8's first bullet, add that batten also releases its held record once the resumed child stops again without having taken the record back. Add the matching case to the acceptance criterion on the PR-Gate hand-over.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:10 (Scope)
    summary: The board's open point on how batten's Land row takes the lock is answered only in support-log.md (Interview turn 3, Q4), which Plan-Write never reads; the decision record answers the other two open points but not this one
    fix: Add a Scope Out bullet saying batten's Land row takes no parent lock. Finalize has already landed and released the lock before Land, and Land only pulls the prime, deploys and prunes under its existing land lock.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:98 (D6), with Acceptance criteria at line 218
    summary: D6 and an acceptance criterion name a review cut short by a reroute, but reroutes leave only Publish, PR-Gate and Finalize, which all run after Merge-Bouncer's Done, so no reroute can cut a merge review short
    fix: In D6, replace "a review cut short by a reroute records nothing" with "a review that ends without Merge-Bouncer's Done, such as an escalation followed by a goto past it, records nothing". Reword the acceptance criterion at line 218 to the same case.
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:132 (D9)
    summary: The reroute cap counts three per run without a landing, but D9 does not say whether the blocked halt's own way forward, lyx loom resume, resets the count, so a resume may re-halt at the next parent move with no way to grant more reroutes
    fix: In D9, state what span the reroute count covers. For example, it counts from the row's last Done or from the last resume of a reroute-cap halt. Also state what lyx loom resume does to it. Add that case to the acceptance criterion on the fourth reroute.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:26 (Scope Out), with line 17 and line 197
    summary: Darn having no merge perch is stated three times, in Scope In, Scope Out and Auto-mode assumptions
    fix: Delete the Scope Out bullet "A merge perch in darn." Scope In already says darn gains Merge-In without the merge perch, and the Auto-mode assumption keeps the reason.
---

### [BLOCKING] Merge-In and the moved-parent check read different parent refs

**Location:** `_lyx/discussion/decision-record.md:127` (D9), with D1 at line 32

**Issue:** D9 reads "the code side's parent tip from its upstream after a fetch, the tip a hand push reaches" and compares it with the tip the merge record says was merged.
D1 moves today's merge-in into Merge-In, and today's merge-in merges a local branch name: `mergeresolve.Resolve` calls `Fabric.MergeIn(source)` (`internal/mergeresolve/mergeresolve.go:55`), and `fabricengine.MergeIn` merges that branch into the warp checkout with no fetch (`internal/fabricengine/merge.go:117`).
Nothing in the Decisions brings the local parent up to its upstream.
After a hand push to the upstream, D9 sees a moved parent and reroutes, and Merge-In merges the same stale local tip again.
That repeats until the reroute cap halts the run blocked.
The acceptance criterion "A hand push to the parent's upstream is seen as a moved parent" passes, but the hand-push path D9 exists for never completes.
In the other direction, a local parent ahead of its upstream, such as an unpushed commit in the prime, makes the recorded tip differ from the upstream on every check.

### [BLOCKING] The parent lock is held through PR-Rework

**Location:** `_lyx/discussion/decision-record.md:79` (D5), with D9 at line 134

**Issue:** D9 makes PR-Gate first ensure the lock.
`PRGate.Call` routes a matched rejection to its only Stuck exit, which the recipe sends to rework (`internal/landingshed/prgate.go:46`, `:109`), and a rework walks the whole main line again (the loom recipe header).
D5 releases the lock only at a loop stop, at the PR-Gate hand-over and at Finalize's Done.
None of these fires on PR-Gate's edge to PR-Rework, and the loop keeps running, so it keeps owning the lock through Plan-Write, the Webster batches and their reviews.
D7 frees it only if an agent sits waiting past the bound, which a busy rework does not do.
Meanwhile every other run on that parent waits at Merge-In, for no benefit, since Merge-In redoes the merge after the rework anyway.
The board bounds the lock to the span from the merge to Land.

### [MEDIUM] Finalize's merge-in-required retry merges an unchecked parent

**Location:** `_lyx/discussion/decision-record.md:136` (D9)

**Issue:** `Finalize.Call` step 5 (`internal/landingshed/finalize.go:201-219`) re-runs the catch-up merge-in and retries the parent-side merge when the parent moved after the first catch-up.
D9 keeps Finalize's catch-up merge-in after its check and argues that it brings no new code, but it is silent on this retry.
Under the lock no lyx run lands in that window.
A hand commit to the parent, or a land by another run after a `lyx batten lock release` (D11 never stops the running Finalize), is still merged by the retry and pushed, and no merge review covers it.
This is the focus question of whether a merged parent commit can escape review, and this path lets one through.

### [MEDIUM] Release semantics after a PR-Gate resume are unbounded

**Location:** `_lyx/discussion/decision-record.md:117` (D8), with D5 at line 82

**Issue:** D8's first bullet bounds batten's held record by the idle bound "with no decision", or by the child leaving awaiting "for any stop but a resume".
After a resume, neither applies.
D7 does not apply either, since the owner is batten and not the loop.
D8's second bullet does not apply, since the owner is not the child's loop.
The record is freed only when the resumed loop takes it back at PR-Gate.
If the resumed loop stops first, for example on a failed step or a killed loop, the outcome depends on whether D5's exit hook "releases the run's lock" by run identity or by owner process, and the Decisions do not say.
D8 makes its own release a compare-and-release on the owner.
If the exit hook does the same, the lock stays with a live batten indefinitely.

### [LOW] Batten's Land row and the lock are decided only in the support log

**Location:** `_lyx/discussion/decision-record.md:10` (Scope)

**Issue:** The board's first open point asks how batten's Land row and a hand landing take and release the lock.
The decision record answers the hand landing in Scope Out.
The answer for the Land row, that it needs no lock, sits only in support-log.md's Interview turn 3 and Q4, which Plan-Write never reads.
A plan writer reading the board's open list could add a lock acquire to Land.

### [LOW] A review cut short by a reroute cannot happen

**Location:** `_lyx/discussion/decision-record.md:98` (D6), with Acceptance criteria at line 218

**Issue:** D9 declares `on_reroute` only on Publish, PR-Gate and Finalize, and Merge-Bouncer's on_done is Publish, so every reroute follows a finished Merge-Bouncer Done.
The case D6 and the acceptance criterion name is unreachable, and a plan writer would build a test for a state the recipe cannot produce.
The reachable case is a review that ends without Merge-Bouncer's Done, such as an escalation followed by a goto past it.

### [LOW] The reroute cap's span and its reset are unstated

**Location:** `_lyx/discussion/decision-record.md:132` (D9)

**Issue:** "capped at three per run without a landing" leaves the counting span open, and the fourth reroute's way forward is `lyx loom resume`.
If the count persists across that resume, the next parent move re-halts at once, and nothing lets a human allow more reroutes on a busy parent.
If the resume resets it, the Decision should say so.

### [NIT] Darn's missing merge perch is stated three times

**Location:** `_lyx/discussion/decision-record.md:26` (Scope Out), with line 17 and line 197

**Issue:** Scope In ("without the merge perch"), Scope Out ("A merge perch in darn.") and Auto-mode assumptions all state it.
The Scope Out bullet adds nothing that Scope In does not already carry.

## Rejected

- D2 and D3 zero values: absent keys give the `review_max_bounces` budget, no auto-accept and the review on, and D3's escalation key is a recipe row key that is independent of loom.yaml. This matches `internal/loomengine/config.go`'s existing per-perch fallback, which reads 0 as `review_max_bounces`, and no Decision disagrees.
- The D5, D9 and D12 lock-wait rules agree. Merge-In and the D9 rows wait in one queue with no time limit, a pause halts either one blocked, a waiter's dead ticket dies with its step process, and a reroute keeps the loop's lock, which Merge-In takes back through D4's same-run rule.
- D9's reroute ending "its target's segment" episode: Merge-Review's episode already ended at Merge-Bouncer's Done, so a rerouted run reaches the next merge review on a fresh budget, as `episodeStuckCount` counts.
- D4's same-run reacquire, D7's idle release and D11's force release each state their bound and what they let through.
- A PATTERN scan found no violated constraint in the Decisions. The relevant entries are hub-wide config, told geometry, the wait mark, no denied recovery, daemon wakeups and fabric git.
- No content in Constraints, Auto-mode assumptions or Open risks is deliberation that should move to support-log.md. Each item carries a requirement or a pointer.
