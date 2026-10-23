---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:173
    summary: D10 guards only batten's idle release, but the loop's own idle-release wake hook also frees the parent lock mid-Finalize, and Finalize never re-checks it before landing
    fix: In D10, replace the sentence naming only battenshed's loopLockStep with one stating that no idle release frees the parent lock while its owner's run is at Finalize and running or a verify marker for the run is live, applied in both idle-release paths, batten's loopLockStep and the loop's own wake hook (internal/loomcli/looplock.go's loopIdleRelease/idleReleaseOn), and add `internal/loomcli`'s loop idle-release hook to Scope's In list beside the idle backstop's Finalize check; extend the acceptance criterion at line 271 to say the lock is kept with the fixer session waiting through the gate run, under both the loop's wake hook and batten's backstop.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:48
    summary: D2's merge of the upstream tip, never the local tip, and of the tip read under the push-skip lock, cannot be done through fabric's MergeIn, which takes a branch name and merges the local tip whenever it is ahead of or diverged from upstream
    fix: In D2, record the fact with its evidence (internal/fabricengine/mergeguards.go resolveMergeSources and pickMergeSourceSHA pick the local SHA unless it is a strict ancestor of origin's, and MergeIn re-resolves the name at merge time) and decide the mechanism - fabric's merge-in gains a way to merge a given parent SHA, which Merge-In passes as the upstream tip it fetched or the local tip it read under the lock - and state that Merge-In fetches through its own step so a fetch failure is Stuck; add internal/fabricengine's merge-in to Scope's In list.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:189
    summary: D11's claim that only the orch's lyx loom resume lifts the landing-stop and reroute-cap halts is false under the LLM driver, whose stencil re-launches the loop at any blocked stop after a build change
    fix: In D11, add that the shed driver stencil (contracts/stencils/shed/shed-template-driver.md, the binary_changed re-launch rule) excludes blocked stops whose reason starts with the reroute-cap or landing-stop prefix and hands them back, list that stencil in Scope's In list, and extend the acceptance criterion at line 269 to the LLM driver.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:226
    summary: D14 has lyx config reconcile rewrite the hub's profiles.yaml, overturning the profiles module's SeedOnly, never-rewritten contract without saying so, while PrepareRunSettings already migrates profile documents at read time
    fix: In D14, drop the clause that reconcile applies the migrations to the hub's profiles.yaml and state instead that profiles and run_settings are migrated at read time, since checkProfiles and LayerRunSettings run each module's Migrate through PrepareRunSettings (internal/configreg/runsettings.go), with each line logged at Warn; in the acceptance criterion at line 281 remove profiles.yaml from the files reconcile rewrites and add that a profile holding webster_max_bounces and merge_max_bounces still validates and seeds.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:163
    summary: D10's kind cap on an unreviewed Finalize fix names only Go's _test.go, so on a C# hub, which lyx supports, a fix editing a test file passes the cap and lands with no perch
    fix: In D10 step 4, replace touches a `_test.go` file with touches a test file of the hub's language, and name where the per-language test-file patterns are declared, beside planparser's per-language sourceExtensions table (internal/planparser/reviewclass.go), with a language the table lacks treating every changed file as a test file so the cap over-covers; add a C# test-file row to the acceptance criterion at line 275.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:208
    summary: D13's bound that every deployed landing had the Code-Perch or Darn-Review as its last review is false at review budget 0, used by the hub's docs profile, where the deploy runs code no review saw
    fix: In D13, restate the deploy bound - a run whose perch budget is 0, such as the hub's docs profile at darn review_max_bounces 0, deploys code with no review at all, bounded only by the landing gate, D10's caps and the operator's deploy_branches list - so the removed PR-Gate guard's real bound is stated.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:121
    summary: D7 builds PR-Review on a switched producer of internal/shedadapters that does not exist; the only switch-gated producer is CommandProducer, which runs an operator shell command
    fix: In D7, replace the switched-producer sentence and its rationale with the decided build - a new PRReview engine in internal/landingshed registered in shedrecipe's one map, reading its row's pr_review setting through Env.Setting, Done without acting off, Done after one Info log on - and state that it is no Command row, so PATTERN-command-row does not apply because it runs no command.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:290
    summary: The declared PATTERN-recipe-owns-failures text allows only a fix by the row's own fixer or a halt, but D10 reroutes a loom conflict, a large merge, a spent cap and an over-cap fix to another row
    fix: Change the declared PATTERN-recipe-owns-failures text to - Failing a merge or test at Merge-In or Finalize - the row's own fixer fixes it, Finalize routes back through the recipe's reroute edge, or the run halts to the orch; no driver, batten, arbiter or record target routes it to another row.
  - id: F9
    severity: LOW
    class: scope
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:187
    summary: The reroute-cap prefix is shed-wide, so D11's hand-back row, arbiterstop refusal and batten re-step skip also catch Webster's plan-conflict reroute-cap halt, a change to an out-of-scope row the record does not state
    fix: In D11, limit the reroute-cap prefix match in the driver row, the arbiterstop refusal and batten's re-step skip to a stop whose producer is Merge-In or Finalize, or state that Webster's reroute-cap halt is handed back too and add it to the acceptance criteria at lines 268 and 269.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:160
    summary: D10 step 4 runs the landing gate at Finalize with no tree-scope clause, while D8 keeps darn's tree-scoped command only at Darn-Write and Darn-Burler-Fix, so a docs-only darn Finalize would run the full landing call under the lock
    fix: In D10 step 4, add that a darn Finalize over a diff wholly under tree_scope runs the tree-scoped command in place of the landing call, as D8 states for the darn writer and fix rows, and add Finalize to D8's tree-scope clause.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:83
    summary: merge_fix_attempts counts gate failures in one row entry, which gives one fixer turn at default 2, while D10 counts finalize_fix_attempts as fixer turns
    fix: In D4, state merge_fix_attempts as the fixer turns one row entry may spend, the same unit as D10's finalize_fix_attempts, and state that an escalate verdict spends no attempt, so the resumed Merge-In after lyx loom decision add runs the fixer again.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:164
    summary: The no-review halt keys on code_max_bounces 0, missing a run whose Code-Perch budget is 0 through the review_max_bounces fallback
    fix: In D10 step 4, replace a loom run at `code_max_bounces` 0 with a loom run whose Code-Review perch's effective budget is 0, its own code_max_bounces or the review_max_bounces it falls back to.
---

### [BLOCKING] The loop's own idle release frees the lock mid-Finalize

**Location:** `_lyx/discussion/decision-record.md:173`, D10, and the acceptance criterion at :271.
**Issue:** D10 rests on the board's promise that the parent cannot move between the fix and the landing.
It adds a Finalize check only to batten's `loopLockStep`.
Every loom loop also arms `loopIdleRelease` (`internal/loomcli/looplock.go:136-185`, wired at `internal/loomcli/arm.go:332`).
That hook releases every lock record the loop owns once no agent is busy and one has waited past the idle bound (`internal/parentlock/idle.go:43-52`), because "the run keeps running, and its next lock-taking row reacquires".
Under D10, the re-prompted fixer session sits waiting while the `landing` gate (`[0, 1, 2]`, up to 30 minutes a tier) runs, so the hook frees the lock.
Finalize takes the lock once and does not re-check it before landing, and no later row reacquires it.
Another run's Finalize can then land in between.
Four reviewers found this independently: three lenses and the handler.
Round 1's F10 raised the same mid-block release for batten's backstop alone, and its fix covered only that path; this is the path the fix left open, not a regression.

### [MEDIUM] D2's upstream-tip merge rests on a merge verb that picks its own tip

**Location:** `_lyx/discussion/decision-record.md:48-49`, D2, and Scope :15.
**Issue:** `Fabric.MergeIn(source)` takes a branch name (`internal/fabricengine/merge.go:131`).
`pickMergeSourceSHA` merges origin's SHA only when the local branch is a strict ancestor of it, and the local SHA otherwise (`internal/fabricengine/mergeguards.go:57-59, 74-90`).
A local parent that is ahead of its upstream, holding another run's unpushed squash, is therefore merged: the case D2 exists to exclude.
On the push-skip path, the name is resolved again after the lock is released, so the merged tip can differ from the tip read under the lock.
The fetch inside the verb is best-effort, logging only a Warn (`mergeguards.go:54-56`).
D2 leaves this fact for Plan to discover, and Scope names no `fabricengine` change.
The acceptance criteria at :262-263 would catch it only at test time.

### [MEDIUM] The LLM driver re-launches the landing halts after a build change

**Location:** `_lyx/discussion/decision-record.md:189`, D11, and the acceptance criterion at :269.
**Issue:** `driver_kind: llm` remains selectable per hub, module, run or `--driver` (`internal/sheddriver/template.yaml:1`, `internal/loomengine/template.yaml:44`).
That driver's stencil re-launches on its own when `last_step.binary_changed` is true at "a `blocked` stop" (`contracts/stencils/shed/shed-template-driver.md:95-97`).
A Merge-In escalation or a reroute-cap halt under that driver is re-stepped after any deploy.
That spends the Opus fixer again on an unanswered escalation, which D11's rationale (:192) rules out.

### [MEDIUM] D14 rewrites the seed-only profiles.yaml

**Location:** `_lyx/discussion/decision-record.md:226`, D14, and the acceptance criterion at :281.
**Issue:** The profiles module is registered `SeedOnly` with no `Migrate` (`internal/configreg/configreg.go:132`).
Its doc says the hub reconcile "never rewrites a present one" (`internal/runprofile/doc.go`), and the file's header calls it operator-owned.
D14 changes that contract without saying so.
The rewrite is also unneeded: `checkProfiles` and `LayerRunSettings` run each module's `Migrate` over every profile and `run_settings` document through `PrepareRunSettings` (`internal/configreg/runsettings.go:68-69, 96-99, 137-145`).
Only D14's Warn logging of the migration lines is new.

### [MEDIUM] The Finalize kind cap is Go-only

**Location:** `_lyx/discussion/decision-record.md:163, 176`, D10 step 4.
**Issue:** D10 says the caps are the only bound on an unreviewed Finalize fix.
The kind cap names `_test.go`, but lyx serves C# hubs too (`internal/planparser/reviewclass.go:10-13`, `"csharp": {".cs"}`).
On such a hub, a 30-line fix that weakens an assertion in `FooTests.cs` passes both caps and lands with no review.

### [MEDIUM] D13's deploy bound is false at review budget 0

**Location:** `_lyx/discussion/decision-record.md:208`, D13.
**Issue:** The hub's `docs` profile sets darn `review_max_bounces: 0` (hub `profiles.yaml`).
D9's rewrite then skips Darn-Review, and a loom perch at 0 settles at once (`internal/loomrecipe/doc.go:11, 23`).
Such a landing into a `deploy_branches` branch deploys code no review saw, where PR-Gate stood before.
D13 states the bound as "the Code-Perch, or Darn-Review, as its last review", so the removed guard's real bound is unstated.

### [MEDIUM] D7 rests on a producer that does not exist

**Location:** `_lyx/discussion/decision-record.md:121, 126`, D7.
**Issue:** `internal/shedadapters` has no switched producer.
The only switch-gated one is `CommandProducer`, which runs an operator shell command read through `ExecutableSetting` (`internal/shedadapters/command.go:1, 22-44`) and falls under `PATTERN-command-row`.
The stub runs no command, so the rationale's premise is false, and the choice of engine is left to Plan.

### [MEDIUM] The declared PATTERN-recipe-owns-failures text omits D10's reroute

**Location:** `_lyx/discussion/decision-record.md:290`, against D10 :153, :157, :162 and the declared `PATTERN-parent-lock` at :289.
**Issue:** The declared text allows only "the row's own fixer fixes it or the run halts to the orch".
D10 reroutes a loom conflict, a large merge, a spent cap and an over-cap fix to Merge-In or Darn-Bouncer.
A reviewer held to the declared text would flag D10's own design.
The two declared entries also disagree, since `PATTERN-parent-lock` names the reroute.

### [LOW] The reroute-cap hand-back also catches Webster

**Location:** `_lyx/discussion/decision-record.md:187-189`, D11.
**Issue:** `internal/shedengine/reroute.go:67` writes the "reroute cap reached" prefix for every row.
Webster keeps `on_reroute: Plan-Burler-Review` (`contracts/recipes/loom-recipe.yaml:361`).
D11's prefix match therefore also changes the routing of Webster's plan-conflict reroute-cap halt, which Scope :29 keeps out, and the record does not say so.

### [LOW] Darn Finalize loses the tree-scoped command

**Location:** `_lyx/discussion/decision-record.md:160`, D10 step 4, against D8 :130.
**Issue:** Today all three darn landing sites run the tree-scoped command for a tree-scoped diff (`pattern/PATTERN-verified-tree.md`).
D8 keeps it only for the writer and fix rows, and D10 runs "the `landing` gate" at Finalize.
A docs-only darn diff would then run the full `[0, 1, 2]` call under the lock.
The tree-scoped pass recorded no landing-call entry, so the skip D8's rationale (:135) relies on cannot fire.

### [LOW] The two fixer caps count different units

**Location:** `_lyx/discussion/decision-record.md:83`, against :175 and the acceptance criterion at :266.
**Issue:** `merge_fix_attempts` counts "gate failures in one row entry", so at default 2 the fixer gets one turn and is never re-prompted.
D10 counts `finalize_fix_attempts` as "fixer turns".
Whether an `escalate` verdict spends an attempt, and so whether the resumed Merge-In at :82 can run the fixer again, is also unstated.

### [LOW] The no-review halt keys on a literal key

**Location:** `_lyx/discussion/decision-record.md:164`, D10 step 4.
**Issue:** An absent per-perch budget takes `review_max_bounces` (`internal/loomrecipe/loomrecipe.go:137-151`).
A run with `review_max_bounces: 0` and no `code_max_bounces` reaches no review either, yet the literal condition reroutes it through a zero-round perch.

## Rejected

- No fork finding was rejected. The duplicate reports of the idle-release, MergeIn, LLM-driver, profiles, switched-producer and cap-unit defects across lenses are merged into the findings above.
