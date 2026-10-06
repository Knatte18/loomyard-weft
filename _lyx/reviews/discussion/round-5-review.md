---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:131
    summary: Decision 13 does not bound a merge to one tier, so merging an integration test into a tmux scenario silently moves it to a tier that card gates do not run
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:134
    summary: Decision 13 drops the fresh-fixture-per-test isolation but its restore bound names no observable state and no check, so what a step may leave behind is unbounded
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: Decision 13 timing rule leaves spread, the measured time, the load tolerance and the revert order open, so noise can still decide a merge
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:84
    summary: Decision 8 alone-rerun omits -count=1 that Decision 13 uses and does not say a zero-match run is no verdict, so a rerun can pass vacuously
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: Decision 4 per-subtest t.Parallel rule, Decision 13 serial steps and Decision 2 host-table wording do not say which governs a scenario or whether a protected test may host one
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:105
    summary: Decision 10 reason class says no fold applies without naming a merge, and Goal lists three activities but not the scenario merge
---

### [MEDIUM] Scenario merge is not bounded to one tier

**Location:** `_lyx/discussion/decision-record.md:131`
**Issue:** Decision 13 covers "several integration or tmux tests".
It never says a family stays within one tier.
`PATTERN-test-speed` puts a test in the lowest tier its substrate needs, and "a test needing two substrates takes the higher tier".
A scenario that absorbs an `integration` test into a `tmux` file raises that test's tier.
The card gates run `integration`; `tmux` runs under the plan verify and by hand.
So the merge can silently take a test out of the gate that ran it, with coverage unchanged and nothing flagging it.
Decision 5 handles the opposite direction, and nothing handles this one.
**Fix:** State in Decision 13 that a family is merged only among tests of one tier, and that a merge never raises a test's tier.

### [MEDIUM] The restore bound for merged steps has no observable definition

**Location:** `_lyx/discussion/decision-record.md:134`
**Issue:** Each former test ran on a fresh fixture, so no test could see another's leftovers.
A merge downgrades that guard.
Decision 13 states a bound, "read-only on it or restoring it before it returns", but gives no account of what counts as the fixture's state or how restoration is judged.
A hub or tmux fixture carries files, refs and commits, worktree contents, sessions and panes, and status or lock files.
A step that restores one of them and leaves another, such as history, a reflog entry or a pane, satisfies the sentence as written, and a later step can observe it.
A comment naming the mutation would also satisfy a reader.
**Fix:** In the first bound, define the fixture's state as everything a later step can observe, and say a step that cannot return all of it to what it found is not merged.
Add that a restoring step asserts the restored state, so a leak fails the scenario instead of hiding.

### [MEDIUM] The per-family timing rule leaves four inputs open

**Location:** `_lyx/discussion/decision-record.md:143`
**Issue:** The rule states the command, package, `-count=1` cache handling, median of three, a tie outcome and a load record.
Four things stay open.
"The spread of the before runs" is undefined: range, standard deviation or something else.
The measured time is not named: the wall time of the `go test` command, which includes build and link after the merge edit, or the elapsed time `go test` reports for the package.
Load is recorded but nothing says when a run is discarded or repeated because the before and after loads differ.
"Taken back to back" does not say whether the three before runs precede the three after runs or the runs alternate, and the before runs need the old tests, so the order of merge, measure and revert is implicit.
Any of these lets noise or a one-off build cost decide a merge.
**Fix:** In the timing bound, define the spread as the range of the three before runs, name the reported test elapsed time as the measure, set the load-difference rule, and state that the before runs are taken on the unmerged tests, then the merge is made, then the after runs, and a tie reverts the merge.

### [LOW] Decision 8's alone-rerun can pass vacuously

**Location:** `_lyx/discussion/decision-record.md:84`
**Issue:** Decision 13 sets the order: a `failed` entry gets its verdict under its base name before it may be merged, and afterwards the scenario is re-run as a whole.
That settles which name is re-run and when.
Two gaps remain.
The Decision 8 command has no `-count=1`, while the Decision 13 command does, so a cached pass can stand as the verdict.
Neither Decision says that a run matching no test, as `-run '^Name$'` does once the test is gone or renamed, is not a verdict.
**Fix:** Add `-count=1` to the Decision 8 command, and state in Decision 8 that a run reporting no tests to run is not a verdict.

### [LOW] Decisions 4, 13 and 2 do not say which governs a scenario

**Location:** `_lyx/discussion/decision-record.md:58`
**Issue:** Decision 4 says a folded table runs `t.Parallel` per subtest unless a comment names the global state.
Decision 13 makes scenario steps `t.Run` subtests that never call `t.Parallel`.
The two are consistent only because Decision 4 speaks of folded tables, and nothing says a scenario is not one.
Decision 2 says "a fold or merge may use [a receiving test] as the host table, under its own name", while Decision 13 says a protected test stays "under its own name", which can be read as not merged at all.
**Fix:** In Decision 13, say that Decision 4's per-subtest `t.Parallel` rule applies to folded tables only and that scenario steps are serial.
In Decision 13, say whether a protected test may name a scenario that hosts other steps, or reword Decision 2 so a merge does not host.

### [LOW] Decision 10 and Goal omit the merge

**Location:** `_lyx/discussion/decision-record.md:105`
**Issue:** Decision 10's last reason class reads "for a `Removable: no` candidate that no fold applies to".
Decision 2 lists a merge as an outcome next to a fold, but a candidate that no merge applies to, because its family failed the timing rule, is not named, and the reason class does not cover a merge that was ruled out.
The Goal lists deleting, folding and tier moves and omits the scenario merge, although Decision 13 and an acceptance criterion carry it, and the Goal is the plan writer's first read.
**Fix:** Change Decision 10 to "no fold or merge applies to", and add the scenario merge to the Goal's list of activities.

## Focus departures

None.
Directive 2 asks whether a comment naming a mutation is enough to merge; Decision 13 does not say so, and F2 records the gap in the bound instead.
Directive 5 is settled as not a defect for Decisions 2 and 12, which name the merge, and is recorded under F6 only for Decision 10.
