---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 (three conditions and Bound)
    summary: D3's HEAD condition cannot see uncommitted writer edits, so a dirty tree reroutes to Merge-In, whose dirty-tree way forward ships unreviewed edits past the perch and Describe or loops back through the reroute
    fix: In D3, add a fourth condition to the bulleted list, that the task worktree has no uncommitted changes, read through the same told seam style, with a failed status read counting as dirty so the writer runs. Replace the Bound sentence "the HEAD condition makes that safe" with one that names both the HEAD and the clean-tree conditions as what makes the skip safe. Add to the Acceptance criteria list of writer-spawning cases "a dirty task worktree".
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 (owned conditions)
    summary: The record's tests hold only test-kind failures, so a log with a build or package failure in an impacted package plus one failing test outside the set classifies as not owned, against every uncertain case counting as owned
    fix: In D1's owned list, replace "the record names no failing test (a build failure, a timeout, or a log the parser could not read)" with "the failed run's log holds any failure that is not a test failure, a build, package-level or unparsed failure, or the record names no failing test", and state that the classifier reads the failure identities of the log the record names, not only the record's tests. Add an Acceptance criterion saying that a log with a build failure in an impacted package and a failing test outside the set writes target Darn-Write.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Goal, D1 Rationale, D2 reason text, D5 Bound
    summary: D5 knowingly classifies a task-caused binary-driving test failure as not owned, while D1 claims the diff provably cannot reach a not-owned package and the Goal says the hand-back calls it not the task's code
    fix: In D1's Rationale, replace "whose failing package the diff provably cannot reach through the import graph" with "whose failing package the import graph shows outside the diff's reach; a test that drives the lyx binary can still reach the change (D5)". In the Goal, replace "with a way forward that says it is not the task's code" with "with a way forward that says it lies outside the task's impacted set". In D2, add that the not-owned reason, for a failing package whose tests import internal/testkit/lyxbin, says the test drives the lyx binary and may reach the task's change, so the Darn-Write goto is the way forward when it does not fail on the parent.
  - id: F4
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3 (third condition and Bound)
    summary: D3 counts the upstream tracking ref as a parent tip, but Merge-In under push skip merges only the local tip, so the tip stays outside HEAD and every later Darn-Write entry reroutes again until the reroute cap
    fix: In D3's third bullet, state that the tip read is the tip Merge-In would merge, the local parent branch only when the run skips pushes, and the local branch and its upstream tracking ref otherwise, so the Bound sentence "After Merge-In merges, the parent tip is an ancestor of HEAD" holds in both modes.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 Bound, sentence beginning "When Merge-In is Stuck without merging"
    summary: D3 says each Darn-Write entry reroutes back to a Stuck Merge-In until the parent repairs the pair, but shed's reroute cap halts the fourth reroute since the row's latest Done, and the Stuck list omits a parent ahead of its upstream
    fix: Rewrite that sentence to "When Merge-In is Stuck without merging, on a parent pair that is dirty, mid-merge, ahead of or diverged from its upstream, a later Darn-Write entry reroutes back to it, and shed's reroute cap halts Darn-Write after three reroutes since its latest Done with its reroute-cap way forward."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3; internal/loomshed/darnwrite.go:15
    summary: A gate-exhausted Darn-Write halt still tells the parent that resume re-spawns Darn-Write, but under D3 that resume reroutes to Merge-In when HEAD is unchanged and the parent moved
    fix: In D3, add a sentence that the gate-exhausted halt's resume way forward says the resume re-spawns Darn-Write, or first takes a moved parent in through Merge-In, so the text matches what the resume does.
  - id: F7
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4, last sentence
    summary: D4 names the shared reason helper as the file's writer but no component for its removal, and the helper sees only the failing outcome, with no row or path and no passing call
    fix: In D4's last sentence, state that each gated producer in internal/shedadapters is told its row's findings-file path, writes it through the shared reason helper when its gate is exhausted, and removes it on a call whose gate outcome passed.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D6, Scope In list, last Acceptance criterion
    summary: D6 calls a test case a row in the shell guard's allow table, which the guard does not have, and pins ./internal/x while the round gate renders the record's import path
    fix: In D6 and Scope, replace "the shell guard's existing allow table gains one row" with "TestRows_PatternsDenyAndAllow gains one case; internal/shellguard's rows are unchanged", and in D6 and the last Acceptance criterion replace ./internal/x with an import-path package such as github.com/Knatte18/loomyard/internal/x, the form the round gate renders.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Scope Out item 1, D3 Rationale and Bound
    summary: Deliberation narrative and restated conditions sit in the record beside the requirements they repeat
    fix: Delete the Scope Out sentence beginning "This departs from the 2026-10-22 re-scope" (the Auto-mode assumption already flags the departure, and Q10 and Turn 1 hold the evidence). Delete D3's Rationale sentence "The board's 2026-10-20 case, a failing test outside the branch, is not owned under D1 and never reaches Darn-Write." and move it to the support log's Interview. In D3's Bound, delete "A record left by an earlier run never reroutes," and "A parent that moves only on origin, with no fetch since, reads as not moved, and the writer runs.", which restate the first condition and the third condition with the Auto-mode assumption.
---

### [BLOCKING] D3's skip of review is not safe when the writer left uncommitted edits

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:**
D3's safety argument is that the branch carries no new code since the failed verify, which it infers from HEAD equalling the record's `head`.
`darnWrite.Call`'s `Commit` commits only the landing directory (`internal/loomshed/darnwrite.go:101`, `internal/loomcli/darnwiring.go:166-170`).
That means a writer whose gate is exhausted with uncommitted source edits leaves HEAD at the record's `head`.
If the parent then moves, the resume reroutes to Merge-In.
Merge-In halts on its clean check (`internal/landingshed/mergein.go:161`), and `dirtyReason`'s way forward says "commit or remove them on the task branch, then resume".
From there the parent has two paths, and both are wrong:
- following that way forward ships the writer's edits through Merge-In and Publish with no Darn-Review perch and no Describe;
- a `goto --to Darn-Write` reroutes straight back to the same Stuck Merge-In until the reroute cap.

The bound-breaker and existing-paths lenses each found this.

### [MEDIUM] D1 misclassifies a mixed failure log as not owned

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:**
`failingTestsOf` (`internal/loomcli/landingdeps.go:415-427`) keeps only `FailureKindTest` and drops `package` and `opaque` failures (`internal/websterengine/verifyparse.go:12-14`).
Take a build failure in an impacted package together with a failing test in an outside package.
The record then names only the outside test, so D1's "names no failing test" clause does not fire, and the task's own break routes away from Darn-Write.
The existing-paths, bound-breaker and walkthrough lenses each found this.

### [MEDIUM] D1's "provably" claim and the Goal's hand-back text contradict D5's stated reach gap

**Location:** `_lyx/discussion/decision-record.md` Goal, D1, D2, D5

**Issue:**
D5 states that a binary-driving test reaching a change through a subcommand leaves the impacted set, and that D1 counts its failure as not owned.
That failure can be caused by the task, which is exactly why the whole-tree sites still run it.
D1's Rationale nonetheless says a not-owned package is one the diff "provably cannot reach", and the Goal says the hand-back tells the parent "it is not the task's code".
The parent is then steered toward fixing the parent branch first.
The interactions, walkthrough and bound-breaker lenses each found this.

### [LOW] D3 reads a tip Merge-In will not merge under push skip

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:**
Under `PushSkipped`, `syncParent` returns only `tips.Local` (`internal/landingshed/mergein.go:216-221`).
D3 also reads the upstream tracking ref (`internal/fabricengine/parenttips.go:51-63`).
An upstream ahead of local therefore stays outside HEAD after Merge-In's Done, which breaks D3's no-second-reroute claim.
Publish's own refusal under push skip limits the impact.

### [LOW] D3's Stuck-pair bound omits the reroute cap and the ahead case

**Location:** `_lyx/discussion/decision-record.md` D3 Bound

**Issue:**
`rerouteStuck` halts the fourth reroute since the row's latest Done with "reroute cap reached" (`internal/shedengine/reroute.go:13-67`).
D3's own sentence promises the reroute repeats until the parent repairs the pair.
Its list of Merge-In Stuck causes also omits "ahead of its upstream" (`internal/landingshed/mergein.go:231-232`).

### [LOW] The gate-exhausted halt's resume way forward goes stale under D3

**Location:** `_lyx/discussion/decision-record.md` D3; `internal/loomshed/darnwrite.go:15`

**Issue:**
`darnResumeWayForward` promises that a resume re-spawns Darn-Write.
With HEAD unchanged and a moved parent, D3 makes that resume reroute to Merge-In instead, and the record does not reconcile the text.

### [LOW] D4 leaves the removal of the findings file unowned

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:**
`gateFailedReason` (`internal/shedadapters/gateattempts.go:16-21`) takes only the `*GateOutcome` and runs only on the failing branch (`multillm.go:151`, `burlerfix.go:257`).
D4 names it as the writer of the file.
It gives no component that knows the row's path on a passing call, so the removal rule has no owner.

### [LOW] D6 names a nonexistent guard allow table and pins a form the gate never renders

**Location:** `_lyx/discussion/decision-record.md` D6, Scope, Acceptance criteria

**Issue:**
`internal/shellguard/rows.go` has no allow table.
The table D6 means is `TestRows_PatternsDenyAndAllow`'s case list (`internal/shellguard/rows_integration_test.go:56`).
Read literally, the wording asks for a guard-side allow rule, which would be an unbounded guard weakening.
The reproduce step the round gate renders uses the record's import-path package (`internal/loomshed/publishfailure.go:133`), not `./internal/x`.

### [LOW] Narrative and restatement in Scope and D3

**Location:** `_lyx/discussion/decision-record.md` Scope Out item 1, D3

**Issue:**
I applied the completeness-before-leanness test.
Scope keeps the requirement the plan writer needs, which is item 1 out with no reed change.
The departure justification repeats the Auto-mode assumption and Q10.
D3's 2026-10-20 sentence is deliberation narrative, and two Bound sentences restate D3's own first and third conditions.
The Open risks entries each add a consequence or cost beyond the Decision they cite, so I kept them.

## Rejected

- D4's claim that the Burler review row clears the file is moot rather than wrong, since `BurlerReview` never writes it; it changes no behaviour.
- The interactions lens's D2×D5 entry duplicates F3 and is merged there.
- Board-entry check: no Decision, including the Scope item 1 closure and the re-scope departure, asks the run to change this task's board entry body or scope. Not a defect.
- Item 1's closure itself: `98b171b05` names the exact flake and its root cause, and the record flags the departure for the parent through the Auto-mode assumption and Q10. Not a defect.
- D5's bound on the whole-tree runs holds: `gateCallSource` renders `--all` (`internal/loomcli/roundgate.go:43`), and the record states the tree-scoped exception.
- D2's bound holds: the batten marker, the Go driver's `VerifyRetrySpent` and arbiterstop's target-naming refusal cap the retry at one Merge-In re-entry.
