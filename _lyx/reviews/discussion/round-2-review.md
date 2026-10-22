---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 way forward, D4 refusal, D5 refusal, Constraints
    summary: The D4 and D5 refusals, in bound modules loomcli and sheddriver, name plain git in the parent's code checkout, which refusal-spec's way-forward definition does not admit
    fix: Add to D2 a decision that the task amends contracts/specs/refusal-spec.md's What a way forward is list in the same commit, widening item 3 from plain git in the task worktree to plain git in a checkout the message names, when the work there is the operator's and no lyx verb covers it. State in D2 that the D2, D4 and D5 texts rely on that widened item. Extend the Constraints bullet on PATTERN-refusal-way-forward to name the amendment. Leave the PATTERN and spec edit itself to the plan, as the discussion spec says.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2, the paragraph starting A commit there reaches origin
    summary: A commit in the parent's code checkout moves the parent tip, so the re-stepped Finalize reroutes to Merge-In and never lands with no gate run, contrary to D2
    fix: Replace D2's sentence A commit there reaches origin with Finalize's parent push, with no gate run of its own with the route the code takes. Finalize's step 1b2 (holdParent, then parentGuard.moved in internal/landingshed/parentguard.go) reads a local parent tip that differs from the merge record's merged tip and does not hold the task's work as moved, and Finalize's on_reroute is Merge-In in contracts/recipes/loom-recipe.yaml. So after a commit, goto --to Finalize plus resume reroutes to Merge-In, and the commit is merged into the task branch and passes Merge-Review, Publish and PR-Gate before Finalize lands and pushes it. After a discard, Finalize lands directly. Add an acceptance criterion pinning both, that after an operator commit in the parent's code checkout, goto --to Finalize and resume reroute to Merge-In with no dirty halt, and after a discard, Finalize lands.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2, the merge-in-progress way forward
    summary: The merge-in-progress way forward (let the parent pair's fabric merge conclude) names neither a verb nor how to see the merge, so a stale record wedges the run, against the record's own way-forward Constraint
    fix: Make D2's merge-in-progress way forward name the parked merge, meaning its verb and source from the parent pair's merge record, as premerge.go's in-progress reason does. Have it name lyx fabric merge --continue and lyx fabric merge --abort run in the parent pair by the party that started it, ending with refusal-spec's clause for a role-guarded lyx fabric verb, then lyx loom goto --to Finalize and lyx loom resume. Update the matching acceptance criterion to assert those names.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 dirty arm and acceptance criteria
    summary: When the guard error carries worktree dirty with another aggregated reason (not synced, detached head, source not found), the way forward covers the dirt alone, so the goto it names is admitted and Finalize halts again
    fix: State in D2 that, when the guard error carries reasons beyond worktree dirty and merge already in progress, the stuck reason keeps the error's full text and the path list and the way forward clears the dirt first and then the other named reasons. Add an acceptance criterion with a dirty parent whose guard error also carries another reason, asserting the stuck reason names both.
---

### [BLOCKING] Way forward names plain git outside the task worktree

**Location:** `_lyx/discussion/decision-record.md` D2 (the way forward), D4 (the refusal uses D2's rendering), D5 (same text), Constraints.

**Issue:** `PATTERN-refusal-way-forward` binds `internal/loom*` and `internal/shed*`, so it covers D4's goto-guard arm in `loomcli` and D5's driver check in `sheddriver`.
That pattern says a refusal names its way forward "as `contracts/specs/refusal-spec.md` defines one".
The spec's `What a way forward is` list admits only three kinds: a `lyx` verb, a self-taken transition, and "a plain `git` command in the task worktree, when no `lyx` verb covers it".
D2's way forward is "commit or discard them in `<checkout>` with plain git", where `<checkout>` is the parent's code checkout, so the step happens outside the task worktree.
None of the three kinds covers it.
The record gives sound reasons for naming plain git rather than `lyx fabric commit` (`ScopedPathspec`, `sandboxedRoleRefusal`), but it never squares that choice with the spec.
Its Constraints bullet even cites `PATTERN-refusal-way-forward` as satisfied.
Because the pattern is a listed constraint, this is BLOCKING.
The fix is a recorded decision to widen the spec, since no `lyx` verb clears an operator's tracked dirt.

### [MEDIUM] An operator commit reroutes Finalize to Merge-In

**Location:** `_lyx/discussion/decision-record.md` D2, the paragraph "A commit there reaches origin with Finalize's parent push, with no gate run of its own …".

**Issue:** The focus asked whether a commit there reaches origin with Finalize's parent push.
It reaches origin eventually, but not by the route D2 states.
Finalize's step 1b2 calls `holdParent`, which calls `rerouteIfMoved` (`internal/landingshed/stuck.go`).
`parentGuard.moved` (`internal/landingshed/parentguard.go`) counts the parent as moved when its local tip differs from the merge record's `MergedTip` and does not hold the task's work.
Its own doc says "A local parent ahead of or diverged from its upstream therefore reads as moved."
An operator commit in the parent's code checkout is exactly that state.
Finalize returns a reroute Stuck, and its `on_reroute: Merge-In` row in `contracts/recipes/loom-recipe.yaml` sends the run back through Merge-In, the Merge-Review perch, Publish and PR-Gate before Finalize runs again.
So "with no gate run of its own" is false.
The commit is merged into the task branch, verified and reviewed before it lands.
That reroute is the safe outcome, but the record presents a different one, and nothing in the acceptance criteria pins either arm.
On the focus's other question: D2 does name who performs the step, the operator alone, with the orch relaying and then running the goto and the resume.
That part holds.

### [MEDIUM] The merge-in-progress way forward names no verb and no holder

**Location:** `_lyx/discussion/decision-record.md` D2, "let the parent pair's fabric merge in progress conclude, then `lyx loom goto --to Finalize` and `lyx loom resume`".

**Issue:** The focus asked what text the operator gets when a merge record exists and the parent is also dirty.
`Merge` (`internal/fabricengine/merge.go`) skips the foreign-state check when a record exists.
It then aggregates `mergeInProgressReason` and `pairDirtyReason`, so the error carries both `merge already in progress` and `worktree dirty`, and `WorktreeDirty()` is true.
Finalize's dirty arm therefore fires, and D2 correctly routes it to the error's own text with no path list, while D4 admits the goto.
That much is coherent.
The gap is the way forward itself.
"Let it conclude" names no command and no way to see who holds the merge.
The refusal spec counts waiting only "when the message names how to see the holder".
If the record is stale, for example left by a crashed sibling run or by this run's own interrupted attempt, nothing ever concludes it, and the run halts at Finalize indefinitely.
The record's own Constraint says every stuck reason names its way forward.
The repo already has precedent: `mergeWayForward` in `internal/landingshed/premerge.go` names the parked merge's verb and source, then `lyx fabric merge --continue` / `--abort` with the role-guard clause.

### [LOW] Other aggregated reasons beside the dirt are left to a second halt

**Location:** `_lyx/discussion/decision-record.md` D2's dirty arm, acceptance criteria.

**Issue:** `Merge` aggregates every failed precondition, so a dirty parent can also carry `branch not synced to upstream`, `checkout is not on a branch` or a source reason.
D2's dirty arm keys only on dirty without in-progress, and its way forward tells the operator to clear the dirt and goto.
D4 then admits, since it checks only dirt, and Finalize halts again on the remaining reason.
The error text does name every reason, so nothing is hidden.
But the record never says how the way forward treats them, and no criterion covers the combined case.

## Rejected

- Focus 2 (hand-over at a blocked Finalize stop): not a defect.
  `decideDrive` (`internal/sheddriver/driverules.go`) ranks the hand-over above the tackler rule when the stop is `binaryEligible` and the build changed.
  The successor's `resumeOpenAttempt` (`internal/sheddriver/drive.go`) finds no open attempt, and `run` then calls `Step`, which re-calls the current producer on a blocked run.
  So that route does bypass the goto guard.
  D5 covers it by running a told check at the driver's first status read when the run is blocked with no open attempt, and `handBackRun` writes no status change, so the successor still reads blocked.
  The record states the bound: the check is wired to the Finalize arm alone.
- Focus 3 (whether a Finalize halt's last history entry is Finalize itself): the premise holds.
  In `internal/shedengine/run.go`, a Stuck with `def.OnStuck == ""` calls `appendHistory("")`, which appends the current row's own entry.
  `gotoRows` (`internal/sheddriver/tackler.go`) adds the last entry's producer only when it differs from `CurrentProducer`.
  At a Finalize halt the tackler can name only Finalize, so D4's narrowing to Finalize is sound.
- Focus 5 (refusal class): not a defect.
  The refusal spec's transient class is "a condition that clears by itself or by a named action", disposed as "halt without mutating state; the message names the retry".
  D4's arm fits that: it decides only, under the `GotoGuard` contract, and names the retry.
  It is no policy guard, because the step it blocks would always fail on Finalize's own guard; it does not steer a step that could land.
  The record no longer calls the arm advisory, so the earlier classification gap is closed.
- D1's premise checked: `git ls-files _lyx` is empty in this code checkout, and the shared `.git/info/exclude` holds `/_lyx`, as `wireJunctionSet` (`internal/fabricengine/junction.go`) writes.
- D3's removals checked: `IsWorktreeDirtyReason`'s only production caller is `internal/loomcli/halt.go`, and `haltnote.Note.ParentStatus` is written only there.
