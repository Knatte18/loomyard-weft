---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:134
    summary: Decision 13's merge-eligibility bound is defeated by its own or-clause, since a step that mutates the fixture may be merged if a comment names the change, and no restore or independence requirement remains
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:135
    summary: The stop-on-failure bound assumes serial steps, but Decision 4 requires t.Parallel per subtest, and a parallel step makes t.Run return true before it runs, so the stop is vacuous
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:140
    summary: Decision 13's per-family timing has no margin or tie rule, no load record, no -count=1 and names no package, so under the noise the Open risks name a merge can be decided by noise or by a cached run
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:136
    summary: Decision 13's alone-rerun bullet leaves open how Decision 8 applies, since a merged test name no longer exists for the -run rerun and the order of Decision 8's verdict versus the merge is unstated
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:124
    summary: Decision 12 leaves the statement count (weighted by the profile's statement field, or by block) and the noise criterion unstated, though the directory match and covermode are exact
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:127
    summary: Decisions 2 and 12 predate Decision 13 and omit merging, since Decision 2 lists only fold or keep for a Removable no candidate and Decision 12's rationale names deletion, fold and tier move only
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:42
    summary: The Review rounds placeholder reads No rounds yet directly above a recorded round, and Decision 13's operator-provenance line sits in the decision record instead of the support log
---

### [MEDIUM] Merge eligibility is not bounded
**Location:** `_lyx/discussion/decision-record.md:134`
**Issue:** The bound reads "Only tests whose steps do not change the fixture in a way a later step observes are merged, or whose changes the step's comment names; a step that depends on an earlier step's state names that step in a comment."
The first clause is a real independence test; the `or` clause lets any test merge once a comment names its mutation.
Naming is documentation, not a bound: a step that commits, deletes a branch, or kills a tmux session, and that the next step observes, merges freely.
Each former test ran on a fresh fixture, so this is the one place the merge adds a hidden order dependence, which Decision 8 calls a defect when a test fails alone.
The attack-surface rule applies: the Decision removes per-test isolation and states no bound on what a step may leave behind.
**Fix:** Replace the clause with an eligibility test.
A step leaves the fixture as it found it (read-only on it, or it restores it before returning), and a test whose behavior needs an earlier step's state is not merged.

### [MEDIUM] Stop-on-failure assumes serial steps
**Location:** `_lyx/discussion/decision-record.md:135`
**Issue:** `if !t.Run(...) { return }` stops later steps only when each `t.Run` runs its step to completion.
A subtest that calls `t.Parallel` pauses and `t.Run` returns true at once; the step then runs after the scenario function returns, concurrently with its siblings on one shared fixture.
Decision 4 requires `t.Parallel` per subtest unless a comment names the global state, and Decision 13 says nothing about it, so a plan writer can follow both and get a parallel scenario whose stop never fires.
**Fix:** State in Decision 13 that the scenario's top-level test calls `t.Parallel`, its steps never do, and a comment atop the scenario names the shared fixture as the state that forbids it, which `PATTERN-test-speed` already permits.

### [MEDIUM] The per-family timing cannot decide a merge under load
**Location:** `_lyx/discussion/decision-record.md:140`
**Issue:** A measurement exists that can be taken while editing: `go test -tags integration,tmux -run <the family's tests>`, median of three, before and after.
It is not enough as written.
"Merged only when its own tests run faster" has no margin, so a difference inside the spread of the three runs counts, and Open risks says load alone moves times.
Decision 11 records the load average at its two points; Decision 13 asks for none.
The command names no package, and `-count=1` is absent, so run in package-list mode it reports a cached result.
Before and after are also taken at different moments around an edit, so drift between them is not separated from the merge's effect.
**Fix:** Require `-count=1` and a named package.
Take both medians back to back, record the load average with each, and merge only when the after median beats the before median by more than the spread of the before runs; a tie leaves the tests unmerged.

### [LOW] Decision 8's alone-rerun does not map onto a merged test
**Location:** `_lyx/discussion/decision-record.md:136`
**Issue:** Decision 8 re-runs a `failed` entry as `-run '^Name$'`.
After a merge that name is not a test, and `go test` exits ok with no tests to run, so the rerun passes vacuously.
"Applies to the scenario test as a whole" does not say whether that means the scenario must pass alone, whether a `failed` entry may be merged before its Decision 8 verdict, or how an intentional step-to-step dependence is told from the hidden one Decision 8 hunts.
**Fix:** State that a `failed` entry gets its Decision 8 verdict on its base name before it may be merged, and that after a merge the check is `-run '^<Scenario>$'` passing alone, with `-count=1`.

### [LOW] Decision 12's counting and noise rule are open
**Location:** `_lyx/discussion/decision-record.md:124`
**Issue:** The directory match is exact, `-covermode=set` is named, and test files are excluded, so the three nested packages are settled; a prefix match is ruled out and `internal/gitrepo` does not absorb `internal/gitrepo/internal/gitoracle`.
Left open: "covered statements over total statements" can be summed by each profile line's statement field or counted by block, and the two give different figures for the PR.
Also unstated is that `./...` in `-coverpkg` resolves from the repository root.
The noise exemption says "two runs of the unchanged tree" without saying that the unchanged tree is the base commit, which the after state requires re-running.
**Fix:** Say the figure sums the statement field of the profile's blocks, per file in the package's own directory, with the run started at the repository root, and that a noise claim needs a second run of the base commit.

### [LOW] Earlier Decisions omit merging
**Location:** `_lyx/discussion/decision-record.md:127`
**Issue:** Decision 2 sends a `Removable: no` candidate to a fold or a keep and says "never deleted", with no word on a merge into a scenario, which removes its top-level name too.
Decision 12's rationale says "a deletion, fold or tier move explains no drop" and omits the merge Decision 13 added.
**Fix:** Add the merge to Decision 2's list of paths for a `Removable: no` candidate, or exclude it explicitly, and name merges in Decision 12's rationale.

### [NIT] Support-log leftovers
**Location:** `_lyx/discussion/support-log.md:42`
**Issue:** `_No rounds yet._` stays directly above the "Parent review, round 1" entry.
Decision 13's line "Added by the operator on 2026-10-06, after Discussion" is provenance, with no requirement the plan writer needs, so it belongs in the support log.
**Fix:** Drop the placeholder line once an entry exists, and move the provenance sentence to the support log's Question ledger as a resolved entry for Decision 13, leaving the Decision's content in the record.
