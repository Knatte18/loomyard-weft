---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:52-56
    summary: "The other-reasons way forward puts the dirt step before the checkout-onto-a-branch step, so an operator who commits first commits onto a detached HEAD that no ref reaches."
    fix: "In D2's paragraph on the dirty-worktree reason with another reason (lines 52-56), state the order per reason instead of 'the dirt step first': for `checkout is not on a branch` the step that checks out the parent branch in `<checkout>` comes before the commit-or-discard step, and the commit step is worded to run on that branch. Keep the dirt step first for `branch not synced to upstream`. Add a second example to the combined-case acceptance criterion (line 157), or a sibling criterion, pinning the detached-HEAD ordering: a dirty parent whose guard error also carries `checkout is not on a branch` returns Stuck with the checkout step before the dirt step."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:45-47
    summary: "The merge-in-progress arm offers plain git in the parent's code checkout (commit or set aside any change of the operator's there) but is defined as the error's own text with no path list, and nothing says the reason names the checkout; the widened refusal-spec item 3 admits plain git only in a checkout the message names."
    fix: "In D2's merge-in-progress paragraph (lines 45-47), state that the reason names the parent's code checkout path (the error's checkout field) beside the parked merge, while still carrying no path list, so the plain-git clause `commits or sets aside with plain git any change of their own there` has a named checkout. Extend the third acceptance criterion (line 156) to assert the reason holds the checkout path."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:52-55
    summary: "D2 words the step for each other parent-side reason 'in Merge-In's wording for the same state (syncParent)', but syncParent has no wording for `checkout is not on a branch`; D2 writes that step itself."
    fix: "Reword the sentence at lines 52-53 so the Merge-In wording is cited only for `branch not synced to upstream` (the ahead and diverged cases of `syncParent`), and say the `checkout is not on a branch` step is new text written for this reason."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:65
    summary: "D2 says the plan writes 'the spec and PATTERN edit', but Scope, Constraints and the acceptance criteria name only the refusal-spec amendment, and no PATTERN entry restates the way-forward list (PATTERN-refusal-way-forward points at the spec)."
    fix: "At line 65, drop 'and `PATTERN` edit' so the sentence reads that the plan writes the spec edit; or, if a PATTERN entry does need a change, name the entry and add it to Scope's In list and to the amendment criterion at line 159."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:49
    summary: "The merge-in-progress arm's fallback covers only a record that cannot be read; a parked-merge read that returns no parked merge (the record concluded or removed between the guard and the read) leaves the reason's wording to the implementer."
    fix: "At line 49, widen the sentence to 'When the record cannot be read or the read reports no parked merge, the reason names no verb and source and keeps the same way forward', and add the no-parked-merge read to the third acceptance criterion's fallback case or its own row."
---

### [MEDIUM] Dirt step before the checkout step orphans an operator's commit on a detached HEAD

**Location:** `_lyx/discussion/decision-record.md:52-56`
**Issue:** D2 orders the other-reasons way forward as the dirt step first, then one step per other reason.
For `checkout is not on a branch` the dirt step reads "commit or discard them in `<checkout>`", and the checkout step follows.
An operator who follows that order commits onto a detached HEAD; `internal/fabricengine/mergeguards.go` (`detachedHeadReason`) documents that such a commit is one no ref reaches and that the next checkout discards it silently.
The route the decision describes afterwards (the local parent tip moves, Finalize reroutes to Merge-In) assumes the commit sits on the parent branch, so the order also breaks the route D2 settles.
The acceptance criterion at line 157 pins only `branch not synced to upstream`, so the detached-HEAD ordering has no test.

### [MEDIUM] The merge-in-progress arm's plain git names no checkout

**Location:** `_lyx/discussion/decision-record.md:45-47`
**Issue:** D2 and the focus question rest on the widened item 3 of `contracts/specs/refusal-spec.md`, which admits plain git "in a checkout the message names".
The dirt-only arm and the other-reasons arm render the checkout path.
The merge-in-progress arm is "the error's own text with no path list, followed by the parked merge" and its way forward tells the operator to first commit or set aside with plain git "any change of their own there"; neither the arm's text nor its acceptance criterion says the reason carries the checkout path, and the error's text (`MergeGuardError.Error()`) is path-free by constraint.
Without the path, that clause falls outside the amendment the decision itself makes, so a test that reaches the message cannot pin the widened item.
The decision does cover the widening for the other two arms (Scope, D2, Constraints and the criterion at line 159 all record it), so no further gap exists there.

### [LOW] Merge-In has no wording for a detached parent checkout

**Location:** `_lyx/discussion/decision-record.md:52-55`
**Issue:** `syncParent` in `internal/landingshed/mergein.go` words the ahead, diverged, no-pair, dirty, merge-in-progress and not-fast-forward states; a detached checkout is not among them (no occurrence of "not on a branch" or "detached" in `internal/landingshed`).
The sentence promises Merge-In's wording for every other parent-side reason, so a plan writer searching `syncParent` for the second reason finds nothing.

### [LOW] An unscoped PATTERN edit

**Location:** `_lyx/discussion/decision-record.md:65`
**Issue:** The only record of the widening is `contracts/specs/refusal-spec.md`; `PATTERN-refusal-way-forward.md` defers to that spec and does not list the three items.
The sentence names a `PATTERN` edit that Scope's In list and the acceptance criteria never mention, which leaves the plan writer to decide whether one exists.

### [LOW] No-parked-merge read is not covered by the fallback

**Location:** `_lyx/discussion/decision-record.md:49`
**Issue:** `fabricengine.MidMerge` returns a state of kind none when the record is gone, not an error.
The guard's `merge already in progress` reason is record-based and read earlier, so the record can vanish between the two reads (for example, a `--continue` that concludes meanwhile).
D2 defines the fallback for an unreadable record only, so the reason for a none-state read is unspecified.

## Verified, no finding

- Focus 1: the widening of item 3 is recorded as a decision (D2), listed in Scope, bound in Constraints and pinned by a criterion; the bound "the work there is the operator's and no `lyx` verb covers it" is stated, so the attack-surface question is answered. F2 covers the one arm whose text the widening does not reach.
- Focus 2: D2 states the Finalize re-step route after a commit, and the route holds against the code: `parentGuard.moved` reads the local tip as differing from the merge record's merged tip, `holdParent` runs at step 1b2 before the merge, Finalize's `on_reroute` is Merge-In in `contracts/recipes/loom-recipe.yaml`, and `syncParent` stops on `ParentAhead`. The criterion at line 158 pins both the commit and the discard outcome.
- Focus 3: the merge-in-progress arm names `--continue` and `--abort`, the parked merge's verb and source, the `--abort` hard-reset warning, the `merge conclude already landed` recovery and the role-guard clause, matching `mergeWayForward` in `internal/landingshed/premerge.go` and the `MergeAbort` behavior in `internal/fabricengine/mergelifecycle.go`.
- Focus 4: the combined case is covered by D2's dirt-first paragraph and a criterion; the set of reasons `Merge` aggregates (in progress, dirty, detached, not synced, source reasons) is fully named, so no reason is left without a rule. F1 and F3 concern its wording and order only.
