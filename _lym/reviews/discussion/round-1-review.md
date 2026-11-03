---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md D1 step 3
    summary: D1 calls Add with the run's parent and anchor, so a live pair is refused by Add's anchor-on-live-weft refusal (beam projects, subpath anchors) or by ErrSourceDirty on a dirty prime, and D2 then fails a recoverable run for good
    fix: In D1 step 3, state that the adopt-only Add passes no Anchor and skips the source-dirt check, since an adopted live weft carries its own anchor and the fork point's dirt is not read when nothing is forked (internal/fabricengine/add.go:239-246 and 397-401 are the two refusals). Add a D1 acceptance criterion - a resume of a beam project's run whose task worktree is gone re-adopts its live pair at the anchor its weft records, and one with tracked dirt in the prime re-adopts too.
    origin: lens:bound-breaker
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md D2 Land way forward
    summary: The Land way forward lym fabric add <slug> --adopt-local then lym batten run <slug> does not recover the cases D2 sends it - on the not-live answer the plain add forks a fresh weft over the archived one, dropping the child's run records, and on a permanent refusal it hits the same refusal D1 just got
    fix: Replace D2's Land way forward with one that recovers a landed run whose pair is gone - for the not-live answer, name the step that restores the weft from its archive tag (or a new adopt-only add option over an archived weft, stated with its bound), and for each other permanent refusal class name the step that clears that refusal before the re-run. State how lym batten run treats the failed status at Land on the re-run, and change the D2 Land acceptance criterion to require that following the named steps resumes Land with the child's records present (PATTERN-refusal-way-forward - a test reaches the way forward).
    origin: lens:walkthrough
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lym/discussion/decision-record.md D4 step 2 and Constraints bullet on PATTERN-push-both-sides
    summary: D4 pushes run records from Go without the code branch and with no failed-push record or notice, departing from PATTERN-push-both-sides without declaring a change to it
    fix: Either route D4's branch push through the pattern (code branch along, the failed-push record written and then cleared by Remove's existing end-of-call ClearPushFailure in internal/fabricengine/remove.go) and drop the Notes bullet saying otherwise, or add a changes bullet for PATTERN-push-both-sides under PATTERN changes that carves out a teardown's archive push of a branch it is about to delete, within the 280-character rule-text cap, and make the Constraints bullet cite that declared text.
    origin: lens:bound-breaker
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D1 first paragraph
    summary: The reconcile change is underspecified - with no pair worktree the run scope produces no entry for the slug, so judging it as today returns no owner, ended or stopped verdict, and dropping the shared originTargets error also changes lym adopt's DropLocal and loom's owner and landing reconciles
    fix: Rewrite D1's first paragraph to say the tolerance is an option of the run scope that only batten's resume sets, under which the slug's entry is classified from its origin record and the Bolt without a pair worktree, and every other run-scope caller keeps today's error (internal/fabricengine/originreconcile.go:633, internal/fabricengine/destroy.go DropLocal). Keep the D1 acceptance criteria for the owned-elsewhere refusal and the absent-pair report.
    origin: lens:walkthrough
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D2 first paragraph
    summary: The permanent/transient split is stated by exclusion over Add errors that carry no sentinel, so local operational refusals such as ErrSourceDirty, a worktree-add lock failure or the anchor-move stop land on the permanent side and fail the run for good
    fix: In D2, list Add's refusal classes and the side each lands on - the not-live sentinel, the records-worktree-present refusal and ErrRemoteLeftover as permanent; a transport or probe failure, the D7 lock wait, a worktree-add failure and the anchor-move stop as transient - and state that Add wraps its probe and transport failures in a sentinel the pre-run tests with errors.Is. Add an acceptance criterion for one transient local refusal leaving the status running.
    origin: lens:bound-breaker
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D3 rule, rationale and fabric remove warning
    summary: D3 names lym batten cancel after lym fabric remove for every record not done, Land included, and its rationale misreads childReachedParent - with the child status gone it returns false at Land, so the cancel releases a landed entry, the outcome D2 forbids
    fix: Correct D3's rationale to read that childReachedParent returns true only at Worktree-Teardown or past the parent-side merge (internal/battencli/cancel.go:691-708). Restrict D3's appended cancel and fabric remove's warning to a batten record standing before Land, and at Land name D2's corrected Land way forward instead. Add a D3 acceptance criterion - lym fabric remove of a slug whose batten record stands at Land does not name lym batten cancel.
    origin: lens:interactions
  - id: F7
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D3, Finalize stop in internal/landingshed/finalize_owner.go
    summary: The cancel D3 appends to Finalize's stop lifts the slug-scope owner pause the same stop says to leave standing, or refuses as owned elsewhere when the slug's owner is another machine
    fix: In D3, state what the Finalize stop names after lym fabric remove given that the cancel's LiftPauses lifts every slug-scope pause including the owner pause (internal/battencli/cancelcmd.go liftSlugPauses) and refuses with cancelOwnedElsewhere when the local board shows another owner - either drop the appended cancel from this stop and leave the record to the adopt that clears the owner pause, or keep it and drop leaving the owner pause standing from the stop text. Add an acceptance criterion for the chosen behaviour.
    origin: lens:existing-paths
  - id: F8
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D4 step 1-2 and D6 lease
    summary: D6 leases on the origin tip the deleter read, the tip D4 archived, but D4 step 2 moves origin to the local tip, so read as the step-1 tip every removal D4 pushed for loses its own lease and keeps the branch
    fix: In D6, name the lease as origin's weft tip after D4's branch push - the local tip when that push landed, else the tip D4 step 1 read - and state that D4 archives exactly that tip. Add a D6 acceptance criterion - a removal whose weft tip origin lacked pushes it and then deletes the weft branch on origin under the lease.
    origin: lens:interactions
  - id: F9
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D6, cleanup sweep
    summary: The cleanup sweep archives only the local weft tip and never reads origin's, so D6's lease on the origin tip it reads deletes an origin tip that is ahead or diverged with no archive tag covering it
    fix: Extend D4's archive rule to the cleanup sweep in D6 - the sweep reads the weft branch's origin tip under the slug's teardown lock, archives it as well when it is neither the local tip nor an ancestor of it, and leases the remote delete on it (internal/fabricengine/cleanup.go:243-258, internal/fabricengine/archive.go resolveArchiveTip). Add a sweep acceptance criterion for an orphan whose origin tip diverges from the local one.
    origin: lens:existing-paths
  - id: F10
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md Open risks, both lines
    summary: Both Open risks leave a fact the design rests on to be found later - the 2026-11-03 trigger is handed to the parent's logs although the record admits D6 and D7 may not cover it, and the missing necessary objects cause is left unpinned
    fix: Research both now and record the answers with their evidence in Decisions or Constraints - for 2026-11-03, trace from the code what makes a new pair's weft branch read gone at a pass's bootstrap (internal/fabricengine/pushclassified.go checkOrigin and ErrBranchGone for a branch with an upstream origin lacks, the reconcile's BranchGone verdict, Add's step-11 leased replace of an archived leftover) and read this hub's run logs, then either design for the trigger found or state why D6 and D7 cover it; for missing necessary objects, cite git's receive-pack connectivity check from its docs at the version in use and state which cause D4's branch-first push and retry cover. Then drop both Open risks lines.
    origin: handler
  - id: F11
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D1 Bound
    summary: The bound that D1 never re-creates a pair whose weft tip an archive tag covers fails when a removal leaves the local weft branch behind, since Add reads a local weft branch as live without reading the tags
    fix: In D1's adopt-only option, state that a local weft branch whose tip an archive/<slug>/* tag covers also answers not-live, or reword the bound to name that case as one D1 re-adopts.
    origin: lens:bound-breaker
  - id: F12
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D4 step 3
    summary: Tagging a diverged origin tip needs its objects locally, and D4 step 1 only reads the tip, while resolveArchiveTip fetches only when no local branch exists
    fix: In D4 step 3, state that the diverged origin tip is fetched by SHA before its archive tag is made, as resolveArchiveTip already fetches an origin-only tip.
    origin: lens:walkthrough
  - id: F13
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D5
    summary: Comparing git ls-remote's answer with the commit the stop names reads only the tip, so a commit the remote holds below its tip still reads as absent
    fix: In D5, have the rule read the remote tip with git ls-remote and treat the named commit as present when it equals that tip or is an ancestor of it by git merge-base --is-ancestor where the tip is known locally, and as unsettled otherwise, never absent. Extend the D5 acceptance criterion to cover the ancestor case.
    origin: lens:walkthrough
  - id: F14
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D7 Bound
    summary: D4 lengthens Remove's exclusive hold past the teardown wait Add and the discard share, so a hand lym fabric add or a cancel's discard meeting a removal still retrying its archive is refused, and the record states no bound for it
    fix: In D7's Bound, state the hold D4 can reach (one branch push plus the archive-tag retries, about a minute) against the teardown wait, and either size Add's and the discard's wait to cover it or state that a hand add or discard refused this way names the re-run.
    origin: lens:interactions
  - id: F15
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md Acceptance criteria, first D7 bullet
    summary: The D7 criterion has Add wait on a former pair's detached push holding the lock, but such a push exits at once on finding its worktrees gone, so the criterion pins a momentary state and not the race D7 targets
    fix: Replace the first D7 acceptance criterion with one where Add of the slug waits while Remove, the discard or the sweep of the slug holds the teardown lock exclusive, then makes the pair once it is released.
    origin: lens:interactions
  - id: F16
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md D1 Bound on AdoptLocal
    summary: D1 sets AdoptLocal from an unattended run, which ends the documented rule that only lym fabric add --adopt-local sets it, and D1 does not name the rule it replaces
    fix: In D1's Bound, state that it ends the rule that only the --adopt-local flag sets AdoptLocal (internal/fabricengine/add.go AddOptions.AdoptLocal, internal/fabricengine/doc.go, internal/fabriccli/doc.go) and replaces it with D1's rule that only the adopt-only resume sets it otherwise.
    origin: handler
---

### [BLOCKING] D1's Add call passes inputs that make a live pair refuse

**Location:** `_lym/discussion/decision-record.md` D1 step 3
**Issue:** D1 calls `Add` "with the run's parent and anchor".
Worktree-Create passes the seed's anchor for a project, and `Add` refuses any non-empty `Anchor` when the weft is live (`internal/fabricengine/add.go:397-401`).
With an empty `Parent`, `Add` refuses `ErrSourceDirty` on tracked dirt in the prime (`add.go:239-246`).
D2 reads both as permanent, so the runs D1 exists to recover get a `failed` status instead.
The interactions and walkthrough lenses found the same defect.

### [BLOCKING] D2's Land way forward does not recover the run

**Location:** `_lym/discussion/decision-record.md` D2
**Issue:** On the not-live answer, a plain `lym fabric add --adopt-local` is not adopt-only.
It replaces the archived weft leftover with a freshly forked weft, so the child's run records are gone (`add.go:175-180`, `remoteleftover.go:100-103`).
With the local warp branch gone, it refuses `ErrRemoteLeftover` (`remoteleftover.go:168-175`).
On a records-worktree-present refusal it refuses again (`add.go:328-331`).
The way forward that `PATTERN-refusal-way-forward` requires therefore does not lead anywhere.
All four lenses and the handler found it.

### [BLOCKING] D4 departs from PATTERN-push-both-sides without declaring it

**Location:** `_lym/discussion/decision-record.md` D4 step 2 and Constraints
**Issue:** The weft branch carries run records.
`PATTERN-push-both-sides` makes the code branch go along, and makes a refused push leave a record and a notice.
D4 drops both, and the Constraints bullet dismisses the departure as "a teardown's push" without a declared PATTERN change.

### [MEDIUM] D1's reconcile change yields no entry and reaches other callers

**Location:** `_lym/discussion/decision-record.md` D1
**Issue:** Entries are classified only for pairs whose warp worktree is present (`originreconcile.go:640-652`).
Once the error is removed, the slug gets no verdict, and `resumeRefusal` passes (`resumereconcile.go:113-131`).
The error lives in the shared `originTargets` (`originreconcile.go:633`), which `DropLocal` (reached from `lym adopt`) and loom's owner and landing reconciles also call.

### [MEDIUM] D2's permanent/transient split is not implementable as stated

**Location:** `_lym/discussion/decision-record.md` D2
**Issue:** `Add`'s probe and transport errors are plain wraps (`remoteleftover.go:84-87`, `add.go:415-417`).
Splitting "by exclusion" therefore puts local, fixable refusals on the permanent side.
The interactions lens raised the same point (its F2).

### [MEDIUM] D3 names a cancel at Land and misreads childReachedParent

**Location:** `_lym/discussion/decision-record.md` D3
**Issue:** At Land with the child status gone, `childReachedParent` returns false (`cancel.go:698`).
The cancel D3 names then clears the record and claim of a landed entry, which D2 says must never happen.
The walkthrough and bound-breaker lenses found the same defect.

### [MEDIUM] D3's cancel in Finalize's stop lifts the owner pause or refuses

**Location:** `_lym/discussion/decision-record.md` D3; `internal/landingshed/finalize_owner.go:160`
**Issue:** The stop says to leave the owner pause standing.
The cancel lifts every slug-scope pause, the owner pause among them.
When the local board shows another owner, the cancel refuses as owned elsewhere instead (`cancel.go:512-518`).
The bound-breaker lens and the handler found the same defect.

### [MEDIUM] D6's lease tip disagrees with D4's push

**Location:** `_lym/discussion/decision-record.md` D4, D6
**Issue:** D4 reads O and then pushes L.
A lease on O fails after D4's own push, so the remover keeps the branch and reports that origin moved.
The walkthrough lens found the same defect.

### [MEDIUM] The cleanup sweep has no origin-tip archive for D6's lease

**Location:** `_lym/discussion/decision-record.md` D6
**Issue:** `Cleanup` archives only the local tip (`cleanup.go:245`, `archive.go:140-147`).
A lease on the origin tip it reads can delete an origin tip that no archive tag covers.
Three lenses found this independently.

### [MEDIUM] Open risks defer facts the design rests on

**Location:** `_lym/discussion/decision-record.md` Open risks
**Issue:** The first risk concedes that D6 and D7 may not cover the observed 2026-11-03 failure, and leaves settling it to "the run's mutation records and logs, which the parent may hold".
The code holds candidate triggers the writer did not trace:

- `pushClassified`'s `checkOrigin` refuses a push of a branch whose upstream origin lacks, as `ErrBranchGone`.
- `Add` step 11 replaces an archived leftover on origin.

The second risk leaves the cause of the rejection unpinned, and D4's mitigation is scoped against it.
The rubric counts a deferred research question as a finding.

### [LOW] D1's archive-cover bound misses a surviving local weft branch

**Location:** `_lym/discussion/decision-record.md` D1 Bound
**Issue:** `removeSibling` tolerates a failed local weft-branch delete once the worktree is gone (`remove.go:520-532`).
`probeWeftLeftover` then reads that local branch as live without reading the tags (`remoteleftover.go:88-93`).

### [LOW] D4 tags a diverged origin tip it may not hold locally

**Location:** `_lym/discussion/decision-record.md` D4 step 3
**Issue:** `tagAndPushArchiveTip` runs `git tag` locally (`archive.go:120`).
Step 1 only reads the origin tip and does not fetch it.

### [LOW] D5's rule reads only the remote tip

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** A commit that is an ancestor of the remote tip compares unequal to the tip and is read as absent, which defeats the rule's purpose.

### [LOW] D4's longer hold outruns D7's waits

**Location:** `_lym/discussion/decision-record.md` D7 Bound
**Issue:** The archive-tag retries alone run about 62 s (`archive.go:22-23`), against a 30 s teardown wait.
D7's bound is silent on what a hand caller meets in that gap.

### [LOW] The first D7 criterion pins a momentary state

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria
**Issue:** `DetachedPush` exits as soon as it finds the pair's worktrees gone (`teardownlock.go:94-101`), so the state the criterion describes lasts only an instant.

### [LOW] D1 ends the AdoptLocal flag-only rule without naming it

**Location:** `_lym/discussion/decision-record.md` D1 Bound
**Issue:** `add.go:50`, `fabricengine/doc.go:702` and `fabriccli/doc.go:35` state that only the CLI flag sets `AdoptLocal`, so no unattended run adopts a stale branch.
D1 sets it from a resume without saying that this rule ends.

## Rejected

- Bound-breaker 9 (the cleanup sweep's local half runs outside the lock): D7's rationale concerns origin probes and pushes. A concurrent `Add` that meets the sweep's local delete fails into its own rollback, or the delete gate refuses, so nothing is lost.
- Walkthrough 1's BLOCKING severity: the defect is kept as F4 at MEDIUM. The record states the intent to judge the entry, and its D1 owned-elsewhere acceptance criterion would catch the naive implementation, so it is underspecified rather than wrong.
- The PATTERN-changes entry `PATTERN-slug-remote-order` holds 258 rule-text characters, within the 280 cap (`internal/pattern/check.go:28`). No finding.
- Board rules: no Decision edits this task's board entry, or files or promotes a task. No finding.
