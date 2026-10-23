---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:184
    summary: D10 guards two idle-release paths but batten's owner-pause and owned-elsewhere arms still release the lock under a Finalize fixer, and Finalize never re-checks the lock before landing
    fix: >-
      In D10, replace the sentence that names the two idle-release paths and the sentence that says Finalize takes the lock once and never re-checks it.
      State first that batten's `keepsLockForFinalize` (`internal/battenshed/innerrun_owner.go`), which the owner-pause arm of `lockStep` and `ownedElsewhere` both consult, reads `FinalizePastMerge`, which is true only once the parent holds the task's work, so it releases the lock of a Finalize that is running before its landing.
      Decide whether those two arms keep the lock for a Finalize that is running, which means changing `FinalizePastMerge`, or release it deliberately because the run is paused for another machine; and name that choice and its bound in D10.
      Then add a re-check that holds whichever way that goes: immediately before step 5 Finalize re-reads the parent lock record through its parent guard, and when the record no longer names this run it takes the lock again and re-runs steps 2 to 4 before landing.
      Add an acceptance criterion that a Finalize whose lock record was released while its fixer waited takes the lock again and re-checks the parent before landing.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:52
    summary: D2 says fabric's merge-in gains a way to merge a given SHA but not how the SHA reaches the conflict session, whose resolver and fabric seam take a branch name and sit in a module Scope calls unchanged
    fix: >-
      In D2, name the mechanism.
      The call path is Merge-In to `mergeresolve.Resolve(ctx, source string)` to the `Fabric.MergeIn(source string)` seam in `internal/mergeresolve/deps.go`, so a new fabric method alone never reaches the merge.
      Pick one of two forms and state it.
      Either add a SHA-taking variant of `Fabric.MergeIn` and thread an optional SHA through `mergeresolve.Resolve` and its `Fabric` seam, or let `MergeIn` accept a SHA as `source` and say how `resolveMergeSources` then skips the `origin/<source>` and records-branch derivations and the freshness pick for the warp side.
      Then list `mergeresolve`'s `Resolve` signature and `Fabric` seam in Scope's In list as an interface change only, and reword the Out line for `internal/mergeresolve` to say the conflict session's behavior is unchanged while its entry seam gains the SHA.
      Add the matching test surface to the acceptance criterion that Merge-In over a local parent ahead of its upstream does not merge the local-only commit.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:204
    summary: D11's LLM-driver exclusion of landing halts from the binary_changed re-launch names no way for a recipe-blind stencil to recognise them, and leaves the binary watch in the parking step and the repair guide re-stepping them
    fix: >-
      In D11, state how an LLM driver recognises a landing halt.
      `contracts/stencils/shed/shed-template-driver.md` declares itself recipe-blind and branches only on envelope fields, policy words and error kinds, never on a row name, so a rule naming Merge-In or Finalize cannot live there.
      Have Go mark the halt in what the driver already reads: a field on the stop envelope and on `lyx shed status`, set by the same producer-and-prefix test the Go driver's hand-back row uses, in `internal/shedverbs`.
      Then list every site that re-steps on `binary_changed` and edit each to skip a marked halt: the stencil's Re-launching bullet on `last_step.binary_changed`, its Parking step 3 that starts the binary watch for a binary-change-eligible stop, and the Binary watch paragraph of `contracts/stencils/shed/shed-template-driver-guide.md`.
      Add the guide and the envelope field to Scope's In list, and extend the acceptance criterion on a changed lyx build to say that an LLM driver neither starts the watch nor re-steps after it for a marked halt.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:172
    summary: D10's kind cap tests for a test file of the hub's language but no hub language exists, and darn runs have no plan to read one from
    fix: >-
      In D10 step 4, state where Finalize reads the language the test-file patterns are keyed by.
      lyx hard-codes no language (`internal/languageconfig` holds only skills and rename tools); the only language value is a plan's `language:` key (`go`, `csharp` or `none`, absent read as `go` by `effectiveLanguage` in `internal/planparser`), and a darn run has no plan.
      Decide the rule and write it: for a loom run, the live plan's `language:` with absent read as `go`; for a darn run, the union of every declared language's patterns, or every changed file as a test file; for `none`, every changed file as a test file.
      Say that the new patterns table lacking a language means over-cover, which is the opposite of `sourceExtensions`, where an absent language declares no source file, so the patterns sit beside that table without sharing its default.
      Add acceptance rows for a darn Finalize on a C# hub with a fix that edits a `FooTests.cs` file, and for a `language: none` plan with a small fix to any file.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:226
    summary: D13's deploy bound for a run with review budget 0 omits the Merge-In fixer, whose change has no size or kind cap and no review behind it in a loom run at code_max_bounces 0
    fix: >-
      In D13, correct the bound for a budget-0 loom run.
      D10's size and kind caps bind only Finalize's fixer, while D4's `merge-in` mode fixer may change any file, tests and gate or build configuration included, and D5 leaves its review to the Code-Perch, which a budget of 0 settles approved at once through `zero_rounds` (`internal/loomrecipe/doc.go`).
      Either apply D10's size and kind caps to the `merge-in` fixer's tree diff when the Code-Review perch's effective budget is 0, a change over them halting with the landing-stop prefix and the way forward of D10's budget-0 halt, and state that in D4 and D13; or state in D13 and in Open risks that such a run's merge-in fixer change lands bounded by the `landing` gate alone.
      Reword D13's sentence listing what bounds a budget-0 deploy to match whichever is chosen, and add the matching acceptance criterion.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:135
    summary: D7's reason for a new PRReview engine says the stub has no command to run, but CommandProducer passes on Done for an off switch and for an empty command
    fix: >-
      In the support log's D7 rationale line and in D7, replace the claim that the one switch-gated producer in `internal/shedadapters` runs an operator command which the stub has none of.
      `CommandProducer` (`internal/shedadapters/command.go`) is Done with an Info log when its bool switch is off and also when its command is empty, so a Command row over `pr_review` would be the same no-op.
      State the actual reason a new engine is chosen, that the row is the seat the human PR review engine will replace and must not take an operator shell command, or drop the engine and make the row a Command row under `PATTERN-command-row`.
      Whichever is kept, say in D7 that `PATTERN-command-row` was weighed against it.
---

### [MEDIUM] D10's lock guard leaves other release arms open and Finalize never re-checks

**Location:** `_lyx/discussion/decision-record.md:184`, `internal/battenshed/innerrun_owner.go:109`, `internal/battenshed/innerrun_parentlock.go:40`

**Issue:** The focus asked whether every path that can release the parent lock under a Finalize fixer that waits through the `landing` gate is guarded.
Both idle-release paths D10 names exist and match: the loop's wake hook (`idleReleaseOn` in `internal/loomcli/looplock.go`) and batten's `loopLockStep`, and D10 puts the Finalize check beside each.
Two other batten arms release the same record: `ownedElsewhere` and the owner-pause arm of `lockStep` both call `releaseChildLock` unless `keepsLockForFinalize` holds.
That function reads `FinalizePastMerge`, which returns true for a Finalize only once `ParentHoldsTaskWork` is true, so for a Finalize that is running with its fixer waiting before the landing it returns false and the lock is released.
Today that window is the same, but D10 now makes the long locked wait the common case and says in its own words that Finalize "takes it once and never re-checks it before landing", so nothing stands behind a release the record does not name.
The existing fallbacks, the re-check after a merge-in-required error and the push rejection, catch a moved parent only after the squash has been made on a parent another run may have changed.
The finding is the omission: the record claims the lock is kept and names a subset of the paths that can end it.

### [MEDIUM] D2 leaves the given-SHA mechanism undecided across a module Scope calls unchanged

**Location:** `_lyx/discussion/decision-record.md:52`, `internal/mergeresolve/mergeresolve.go:32`, `internal/mergeresolve/deps.go:24`

**Issue:** The facts D2 states about `MergeIn`, `resolveMergeSources` and `pickMergeSourceSHA` hold: `pickMergeSourceSHA` returns the local SHA unless it is a strict ancestor of origin's, so a local parent ahead of its upstream is merged.
`syncParent` in `internal/landingshed/mergein.go` does read `ParentTips` with a fetch, so the upstream tip is available to pass.
What is open is the path to the merge.
Merge-In merges through `mergeresolve.Resolve(ctx, source string)`, which calls `Fabric.MergeIn(source string)` through a seam in `internal/mergeresolve/deps.go`, and Scope's Out list leaves that module unchanged.
A SHA-taking fabric method cannot reach the conflict session without the seam and `Resolve` changing; passing a SHA as `source` instead sends it through `resolveMergeSources`, which builds `origin/<sha>` and a records-branch name from it.
Scope's In list names only fabric's merge-in, so a plan writer reading it cannot tell which form to build or that `mergeresolve` is touched at all.

### [MEDIUM] D11's LLM-driver exclusion has no recognisable signal and misses two re-step sites

**Location:** `_lyx/discussion/decision-record.md:204`, `contracts/stencils/shed/shed-template-driver.md:97`, `contracts/stencils/shed/shed-template-driver-guide.md:67`

**Issue:** Under `driver_kind` llm the arbiter part holds: the stencil runs the arbiter command at a blocked stop, `arbiterstop.Eligibility` refuses a landing halt with `HandBackWayForward`, and the stencil then hands the stop back.
The `binary_changed` part does not hold as written.
The stencil's header says its body is recipe-blind and branches on envelope fields, policy words and error kinds, never on a row or recipe name, yet a landing halt is identified by producer and reason prefix.
Without a field Go sets, the rule D11 asks for cannot be written in that stencil.
Two more sites re-step the same way and D11 names neither: Parking step 3 starts the binary watch for a binary-change-eligible stop, a blocked stop included, and the guide's Binary watch paragraph re-steps when `binary_changed` is true.
So after a build change the LLM driver would re-step a landing halt through the watch even with the Re-launching bullet fixed, which is the grant of fresh fixer attempts or reroutes that D11's rationale says the hand-back exists to stop.

### [MEDIUM] D10's kind cap names no source for the hub's language

**Location:** `_lyx/discussion/decision-record.md:172`, `internal/planparser/reviewclass.go:10`, `internal/languageconfig/doc.go:3`

**Issue:** The focus asked whether the kind cap covers hubs of every language lyx supports.
The record's own pattern list covers Go and C#, and over-covers a language the table lacks, which is the right direction for an unknown language.
But D10 keys the patterns on "the hub's language", and lyx has no such setting: `internal/languageconfig` holds skills and rename tools and states that lyx hard-codes no language.
The only language value is a plan's `language:` key, with `go`, `csharp` and `none`, and an absent key read as `go`.
A loom Finalize can read the plan; a darn Finalize has none, so for darn the record names no input at all, and falling back to `go` would let a `FooTests.cs` edit on a C# hub pass the cap, the opposite of its purpose.
`sourceExtensions` also treats an absent language as declaring nothing, so a new table beside it needs its own stated default.

### [MEDIUM] D13's budget-0 deploy bound omits the Merge-In fixer

**Location:** `_lyx/discussion/decision-record.md:226`, `_lyx/discussion/decision-record.md:82`, `internal/loomrecipe/doc.go:11`

**Issue:** The focus asked whether the stated deploy bound holds when a run's review budget is 0.
For darn it holds as far as it goes: Darn-Write ends only on a passing `landing` gate, there is no Merge-In, and Finalize's fixer is capped by D10, with D10 step 4 halting where the route back reaches no review.
For a loom run at an effective Code-Review budget of 0 it does not.
The `merge-in` mode fixer aligns and fixes with no line or kind cap, may edit tests and gate or build configuration, and its change is reviewed only by the Code-Perch, which `zero_rounds` settles approved at once at budget 0.
D13 lists the landing gate, D10's caps and `deploy_branches` as the bound, but D10's caps do not apply to that fixer, so its change deploys bounded by the gate alone.
This is a guard that is removed, with the PR-Gate human review gone and the perch off, and the Decision that removes it states a bound that does not cover one of the two fixers.

### [LOW] D7's rationale for a new engine rests on a false premise about CommandProducer

**Location:** `_lyx/discussion/decision-record.md:135`, `internal/shedadapters/command.go:93`

**Issue:** D7 now names a `PRReview` engine in `internal/landingshed`, registered in `internal/shedrecipe`'s one map beside the other landing rows, so the round-2 problem of a producer named in `internal/shedadapters` that does not exist is resolved.
The stated reason for a new engine is that `CommandProducer` "runs an operator shell command, which the stub has none of".
`CommandProducer` returns Done with an Info log when its switch is off and also when its command is empty, so a Command row with `pr_review` as its switch behaves as the stub does.
The decision may still be right, since the row is the seat of a later review engine, but the recorded reason is wrong, and `PATTERN-command-row` says a side-action row is never a new engine.
D7 argues the stub is no side action, which is defensible, but it should rest on the real difference.

## Rejected

- D14's read-time migration of profiles and `run_settings`: `PrepareRunSettings` already runs `module.Migrate` and discards its lines, `checkProfiles` and `LayerRunSettings` both go through it, and the record leaves `profiles.yaml` seed-only and unwritten, so the `SeedOnly` contract of `internal/configreg` is not changed.
- D2's claim that Merge-In reads the upstream tip from its own fetching `ReadParentTips`: confirmed in `syncParent`.
- D11's claim that the reroute-cap prefix is written by shed for every row: confirmed in `internal/shedengine/reroute.go`, so the Finalize-only producer match is needed and sufficient there.
- D10's longest-hold bound and its reroute cap of three: consistent with `rerouteCap` and the two passes it describes.
