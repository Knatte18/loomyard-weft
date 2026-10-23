---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D11 (last sentence), with D4 and D10
    summary: Batten's kept per-build re-step re-steps a reroute-capped Finalize or a Merge-In escalation or spent-cap halt on every new lyx build, granting three more reroutes, a fresh fixer cap or an Opus re-run over an unanswered escalation with no orch turn, which D11's own rationale and the board's cap forbid
    fix: In D11, replace the sentence keeping batten's binary re-step for these halts with one stating that batten's binary re-step skips a blocked stop whose reason starts with the reroute-cap prefix or the landing-stop prefix (a check in internal/battenshed's re-step eligibility beside its blocked-or-failed test), so only the orch's lyx loom resume lifts them; add an acceptance criterion that a changed lyx build does not re-step a Finalize reroute-cap halt or a Merge-In escalation or fixer-spent halt.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D11 (Removed list, internal/arbiterstop's rule)
    summary: Removing arbiterstop's Merge-In and Publish verify-halt refusal while adding the landing-stop refusal only as a sheddriver row leaves the escalation, fixer-spent and reroute-cap halts arbiter-eligible in arbiterstop.Eligibility, which the arbiter verb and batten's eligible-stop hold also answer through
    fix: In D11's Added list, state that arbiterstop.Eligibility refuses a blocked stop whose reason starts with the reroute-cap prefix or the landing-stop prefix, ending with HandBackWayForward, so the arbiter verb, the Go driver's escalation fact and batten's eligible-stop hold all hand these halts to the orch and batten's notice is not held for an arbiter; extend the Go-driver acceptance criterion to the arbiter verb's refusal.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 (Both halts carry the landing-stop reason prefix)
    summary: Only the spent cap and the escalation carry the landing-stop prefix; a fixer timeout past fixer_timeout_min, a missing or unparseable report, a dirty tree after a fixer turn, a failed spawn and a fixer that pushed have no stated outcome and fall to the tackler row, whose resume re-enters the row with a fresh attempt cap
    fix: Add to D4 one sentence stating that every fixer failure exit, at Merge-In and at Finalize (timeout, missing or unparseable report, dirty tree after the turn, failed spawn, a pushed task branch), is a Stuck with Halt whose reason carries the landing-stop prefix and names its way forward, so D11's hand-back row takes it; add the timeout case to the Go-driver acceptance criterion.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 step 4, and Open risks bullet 1
    summary: The finalize_fix_max_lines cap bounds the unreviewed fix under the lock by size only; a one-line t.Skip, a deleted assertion, a build-tag or gate-tier edit passes the landing gate it weakened and lands with no perch, and finalize mode's fix failing tests only is a prompt instruction with no mechanical check
    fix: In D10 step 4, add that a Finalize fixer change touching any _test.go file, any testdata, gate or build configuration, or adding a skip or build constraint, counts as over the cap and reroutes with the commits kept, whatever its line count; state in D10 that this is the bound on what an unreviewed Finalize fix can let through; add an acceptance criterion that a Finalize fixer change under 40 lines that edits a test file reroutes.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D10 rationale (anything larger is reviewed on the way back), with D9's budget-0 rewrite and D5's code_max_bounces
    summary: At a Darn-Review budget of 0 (the hub's docs profile sets it) Finalize's on_reroute is the PR-Review stub, and at code_max_bounces 0 the Code-Bouncer settles at once, so an over-cap or spent-cap fix returns to Finalize unreviewed with fresh finalize_fix_attempts and a fresh 40-line measure, letting up to four passes of fixes and eight Opus turns land with no review
    fix: In D10, state that the 40-line and attempt caps measure from the HEAD at the run's first Finalize entry since the row's latest Done, not from each pass's pre-fix HEAD, and that a Finalize whose route back reaches no review (darn budget 0, or a zero code budget) halts with the landing-stop prefix on an over-cap or spent-cap fix instead of rerouting; correct the rationale sentence to name that exception; add an acceptance criterion for the budget-0 darn over-cap halt.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 (An absent aligned tip ... reads as the task's merge base before this merge), with D3 and D10 step 3
    summary: When the first Merge-In brings no parent commit it spawns nothing and leaves the aligned tip absent; after a large Finalize merge writes merged tip M' and reroutes, the next Merge-In reads the absent tip as the D3 merge base, which is M' itself, sees no parent commit and never aligns what Finalize merged, against D4's own rationale
    fix: In D4, state that Merge-In writes the aligned tip on every Done, the no-spawn path included (set to the merged tip it merged), and that an absent aligned tip reads as the task's fork point from the parent, never a merge base taken from the merge record's merged tip.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 steps 2-3, and D3's reader list naming D10's overlap test
    summary: Step 3 judges the merge large after step 2 merged the parent in, and D3 takes the task side from the merge base of that post-merge HEAD, so the task side holds every file the parent changed and every non-empty loom merge reads as large, while a darn run with no record finds no overlap at all
    fix: In D10 step 3, state that the task side of the overlap is the task's changes from its merge base to the task head before step 2's merge, as the overlap computation the Merge-Review skip seam uses today, and drop D10's overlap test from D3's reader list.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 step 3 and rationale, darn's route back to Darn-Bouncer
    summary: A darn large merge reroutes to Darn-Bouncer so that a perch sees it, but the perch's subject is the task diff at the D3 merge base, which excludes the parent's side, and darn has no aligning fixer, so an overlap-only large merge re-settles or reviews only context drift and nobody reviews the overlap
    fix: In D10, state that a darn large-merge reroute hands Darn-Bouncer the parent's overlapping changes since the previous merged tip as review input for the amendment generation (a Bouncer re-entry seam fed from the merge record), and add an acceptance criterion that a darn overlap-only reroute runs a review whose input names the overlapping parent change.
  - id: F9
    severity: MEDIUM
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 step 4, with D4's fixed or escalate verdict
    summary: D10 states what a spent cap and an over-size fix do at Finalize but not what a finalize-mode escalate verdict does - halt under the lock, reroute, or count as an attempt
    fix: Add to D10 step 4 that a finalize-mode escalate verdict keeps the fixer's commits, releases the lock and halts with the landing-stop prefix, naming the report, with the way forward lyx loom decision add in the task worktree, then lyx loom resume.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D10 (The lock is released on every way out ... batten's idle-release backstop still frees a lock a dead Finalize left)
    summary: Batten's backstop releases a lock owned by another process once every agent of the owner has waited parent_lock_idle_release_min (default 10) with none busy, so the idle fixer session waiting on a long landing gate run (tiers 0, 1, 2, tmux included) lets batten release a live Finalize's lock mid-block, and the parent can move between fix and landing
    fix: In D10, state that the idle backstop never releases a lock while its owner's run is at Finalize and running, or while a verify marker for the run is live (a check in internal/battenshed's loopLockStep beside the idle rule), and correct the dead-Finalize sentence to that bound; add an acceptance criterion that a Finalize gate run longer than the idle bound keeps the lock.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D13 (require_pr_to_base becomes deploy_branches)
    summary: Batten's Land row deploy ran only code PR-Gate reviewed; with PR-Gate and the alignment approve gone, every landing into a deploy_branches branch runs in the prime code that only the Code-Perch reviewed, including Finalize's unreviewed fixer edits, and D13 renames the key without stating what the deploy now lets through or what bounds it
    fix: Add to D13 one sentence stating that the deploy now runs every landing's code with the Code-Perch (or Darn-Review) as its last review and Finalize's capped fix unreviewed, bounded by F4's kind cap and by the operator's deploy_branches list, and that internal/landingshed's doc.go loses its PR-Gate-reviewed claim.
  - id: F12
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D4 (A fixed verdict over a passing gate advances the aligned tip), with D5's answer key
    summary: A fixed verdict over a passing gate advances the aligned tip on the fixer's word alone, and D5's Code-Perch reviews the task diff against the decision record and plan with no view of what the parent brought in, so a misjudged alignment that changed no code is skipped for good and no later row reads those parent commits again
    fix: In D4, state what the advance skips and its bound - the fixer's report names each parent change it judged against the record and its finding - and in D5 add the fixer's latest merge-in report to the Code-Perch's review input, so the perch judges the alignment as well as the code.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 (The lock's longest hold is bounded by ...)
    summary: The stated longest hold omits Finalize's kept retry pass after a merge-in-required error, which re-runs the merge, darn's conflict session and the fixer attempts under the same lock, so the bound can double
    fix: In D10, state that the retry pass reroutes instead of re-running steps 2-4 under the lock, or extend the bound sentence to two passes of the conflict session and finalize_fix_attempts fixer turns.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D14
    summary: D14 puts one table in internal/loomengine's migration yet lists the landing.yaml and gate.yaml moves, which loomengine.MigrateConfig never sees; its order against retiredReviewKeyRenames and copyWebsterModelsOntoMerge is unstated; and configreg.PrepareRunSettings discards Migrate's lines, so the promised Warn has no outlet
    fix: In D14, name each module's own migration for its keys (loom - webster_* to code_*, merge_*, alignment_pr_approve and rework; landing - require_pr_to_base to deploy_branches; gate - the publish list), state that the loom table runs after retiredReviewKeyRenames and copyWebsterModelsOntoMerge, and state that PrepareRunSettings logs each Migrate line at Warn.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 (Only Merge-In writes the aligned tip; the merge record's reviewed-tip field becomes it)
    summary: Merge-In seeds the reviewed tip from the separate reviewed-tip file that RecordMergeReviewed writes, ahead of the record's own field, and D4 leaves that file and its read in place
    fix: Add to D4 that the reviewed-tip file, MergeReviewedPath and RecordMergeReviewed go with the Merge-Review perch, and the aligned tip lives only in the merge record.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D12 (A passing gate at either row removes it), with D10 step 4
    summary: Finalize's gate is skipped over a verified tree, and D12 does not say whether a skip removes the record, so a record a perch fix superseded can stand into Finalize's fixer brief
    fix: In D12, state that a skipped gate over a tree verifytree records as passed removes the record, as a passing run does.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria (Merge-In runs to Done while another run holds the parent lock)
    summary: The criterion is unqualified, but under D2 a push-skipped or upstream-less Merge-In takes the lock for its read and waits behind another run's Finalize
    fix: Qualify the criterion to a parent with an upstream and no push skip, and add one that the push-skipped read holds the lock only for the read.
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D2 and D10 step 1
    summary: Today's Merge-In is Stuck on a parent pair that is ahead of or diverged from its upstream, missing or mid-merge; D2 drops Merge-In's parent-pair checks and D10 step 1 names only the fast-forward of a parent behind its upstream, so where those Stucks now live is unstated
    fix: In D10 step 1, state that Finalize keeps those Stucks under the lock with their ways forward - a parent pair ahead of or diverged from its upstream, missing, dirty or mid-merge - before its merge.
  - id: F19
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Open risks
    summary: The first bullet restates D10's cap, and the second carries the audit's cost figure, deliberation the plan writer needs no requirement from
    fix: Cut the first Open risks bullet once F4 states the bound in D10, and move the 73% and $0.96 figure to the support log's Interview turn 5 entry, keeping the bullet as Merge-In's alignment spawns an Opus session on every merge that brings parent commits in.
---

### [BLOCKING] Batten's per-build re-step reopens every landing halt

**Location:** `_lyx/discussion/decision-record.md` D11, last sentence; D4's halts; D10's reroute cap.
**Issue:** D11 keeps batten's re-step of a halted Go child once per halt and per lyx build.
That re-step spawns the driver through bootstrap, and the only start check is `finalizeGuard`, a parent-dirt check (`internal/loomcli/drivewiring.go:329`, `internal/loomcli/finalizeguard.go:72`).
A re-stepped Finalize past its `RerouteCapped` entry starts a fresh reroute count (`internal/shedengine/reroute.go:13`), and a re-stepped Merge-In is a new row entry with fresh `merge_fix_attempts`.
An escalation is re-run with no decision added.
This hub deploys a new build after every landing, so this happens often.
It contradicts D11's rationale and the board's "a cap on these routes back ends in an ordinary stop to the orch".
The interactions, existing-paths and bound-breaker forks found this independently.

### [MEDIUM] arbiterstop still admits the landing halts

**Location:** D11's Removed list.
**Issue:** `arbiterstop.Eligibility` admits "a halted blocked stop" unless the verify rule D11 removes applies (`internal/arbiterstop/arbiterstop.go:48`).
The arbiter verb, the Go driver's escalation fact and batten's eligible-stop hold all answer through it (`internal/arbiterstop/doc.go:4`).
A row in the driver alone leaves the arbiter verb able to resume an escalation or reroute-cap halt, and batten holds the notice for an arbiter no one runs.
The interactions and bound-breaker forks report the same defect.

### [MEDIUM] Fixer failure exits outside the two prefixed halts

**Location:** D4.
**Issue:** These fixer exits have no stated outcome: a timeout, a missing or unparseable report, a dirty tree after a turn, a failed spawn and a pushed task branch.
Each falls to the tackler's catch-all blocked row (`internal/sheddriver/driverules.go:238`), whose resume grants a fresh cap.
The interactions fork reports the same defect.

### [MEDIUM] The Finalize fix cap bounds size, not kind

**Location:** D10 step 4; Open risks.
**Issue:** No perch reviews Finalize's fixer edit, which `finalize_fix_max_lines` bounds by line count alone.
A test-weakening edit of a few lines passes the gate it weakened and lands.
`finalize` mode restricts the fixer only in its prompt; nothing checks the edit mechanically.

### [MEDIUM] An over-cap fix's route back reaches no review at zero budgets

**Location:** D10 rationale; D9's budget-0 rewrite; D5's `code_max_bounces`.
**Issue:** The hub's `docs` profile sets darn's `review_max_bounces: 0`, so Finalize's reroute goes to the PR-Review stub and straight back.
Each pass measures 40 lines from its own pre-fix HEAD and gets fresh attempts, so the caps bound nothing until the reroute cap.
A loom run at `code_max_bounces: 0` behaves the same.
The walkthrough fork's budget-0 darn loop finding is merged here.

### [MEDIUM] An absent aligned tip skips the alignment of Finalize's merge

**Location:** D4, with D3 and D10.
**Issue:** "The task's merge base before this merge" is, under D3, the merge base of HEAD with the record's merged tip.
After Finalize writes a new merged tip and reroutes, that base is the new tip itself, so the next Merge-In reads no parent commit and spawns no fixer.
Today's field reads as the fork point before any merge (`internal/landingshed/mergerecord.go:42`).

### [MEDIUM] D10's overlap test runs on the post-merge head

**Location:** D10 steps 2–3; D3.
**Issue:** The judgement follows step 2's merge.
Its task side is taken from the post-merge HEAD, which holds the parent's changes, so every non-empty loom merge overlaps.
A darn run with no record sees no overlap at all.
Today's computation uses the pre-merge base and head (`internal/landingshed/mergein.go:288`).

### [MEDIUM] Darn's large-merge reroute shows the perch nothing of the overlap

**Location:** D10 step 3, darn.
**Issue:** The Darn-Review subject excludes the parent's side.
A re-entry reviews only the change of the task diff (`internal/shedadapters/doc.go`, re-entry over a settled verdict), so an overlap-only darn large merge goes unreviewed.
Darn has no aligning fixer to cover it.

### [MEDIUM] A finalize-mode escalate verdict has no outcome

**Location:** D10 step 4.
**Issue:** D4 gives the fixer an `escalate` verdict in both modes, but D10 states only the spent-cap and over-size paths.

### [MEDIUM] The idle backstop can release a live Finalize's lock

**Location:** D10.
**Issue:** `loopLockStep` releases the record once the owner's agents all wait for `IdleBound` with none busy (`internal/battenshed/innerrun_parentlock.go:96`, `internal/parentlock/idle.go:24`).
The fixer session waits while Go runs a full `landing` gate.
The bound-breaker fork reports the same defect.

### [MEDIUM] The deploy loses its reviewed-code bound

**Location:** D13.
**Issue:** `internal/landingshed/doc.go` grounds the deploy in "code PR-Gate reviewed" (`internal/battenshed/land.go:288` gates it).
With PR-Gate gone, Finalize's unreviewed fix reaches the prime's deploy, and D13 states no new bound.

### [MEDIUM] A fixed verdict advances the aligned tip unchecked

**Location:** D4; D5.
**Issue:** Merge-In's alignment is judged by the fixer alone.
The Code-Perch's answer key is the decision record and plan over the task diff, which excludes the parent's commits.
A parent change that contradicts the record without touching task code is therefore never seen again once the tip advances.

### [LOW] The lock bound omits the retry pass

**Location:** D10.
**Issue:** After a merge-in-required error, Finalize retries once under the same lock (`internal/landingshed/finalize.go:245`), which can repeat the conflict session and fixer turns.

### [LOW] D14's single table and its Warn

**Location:** D14.
**Issue:** `MigrateConfig` runs only on loom documents.
`PrepareRunSettings` discards `module.Migrate`'s lines (`internal/configreg/runsettings.go:72`), so the promised Warn is never logged.
The table's order against the existing rename chain is unstated (`internal/loomengine/config.go:757`).
The bound-breaker, existing-paths and interactions forks' matching findings are merged here.

### [LOW] The reviewed-tip file survives D4

**Location:** D4.
**Issue:** `MergeReviewedPath` and `RecordMergeReviewed` keep a second source of the tip that Merge-In reads first.

### [LOW] A skipped gate's effect on the failure record

**Location:** D12.
**Issue:** "A passing gate removes it" leaves a skipped gate undefined.

### [LOW] The lock-free Merge-In criterion is unqualified

**Location:** Acceptance criteria.
**Issue:** D2's push-skipped path takes the lock for its read.

### [LOW] Parent-pair Stucks dropped from Merge-In

**Location:** D2; D10 step 1.
**Issue:** Today's Merge-In is Stuck on a parent pair that is ahead of or diverged from its upstream, missing, dirty or mid-merge (`internal/landingshed/doc.go`, Merge-In paragraph).
The record moves only the fast-forward into Finalize.

### [NIT] Open risks restate D10 and carry a cost figure

**Location:** Open risks.
**Issue:** The first bullet repeats D10's cap.
The second bullet's figure carries no requirement.
The completeness-before-leanness test therefore leaves nothing to extract, and only the figure moves.

## Rejected

- `lyx loom resume`'s refusal text still naming `lyx loom approve` and `reject` (existing-paths): a reader D13's "readers updated" covers; listing it is call-site enumeration, which the rubric's Do-not-flag list excludes.
- D2's push-skipped or upstream-less path merging the local tip after releasing the lock: not a defect. With no push there is no failed push to reset, and the lock held for the read keeps another Finalize from being mid-landing (bound-breaker held it too).
- Own board entry: no Decision changes this task's own entry body or scope. Finalize marks the entry done, which is its status. Scope's Out list leaves other entries to the orch after landing. The Bolt orch rules are not a board entry.
- Deferred research: no Decision, Constraint, risk or criterion leaves a fact for Plan to find. Reconcile over `profiles.yaml`, the merge record's field change and D3's readers are each stated as design, and the gaps found in them are filed above as design findings.
- Relocation of Scope's Out entries and the D4 and D10 rationale: each Out entry tells the plan writer what not to build. Under the completeness-before-leanness test the rationale carries the purpose that F6 and F5 judge against, so none moves.
