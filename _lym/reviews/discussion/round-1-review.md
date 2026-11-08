---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:133 (D7, shipped template sentence)
    summary: The template listing of reports on four keys never reaches a hub whose base gate.yaml already sets those keys whole, as this hub does, so the step never runs here and the goal is unmet.
    fix: In D7, replace the last paragraph's template-only sentence with a Decision on how a hub that tunes webster, parent-align, darn-write and code-burler-fix gets reports added, for example a hubreconcile reinterpretation that appends reports [redundancy] to those keys, or a stated operator step through lym config gate --set filed as a board note, with its bound; cite _bolt/_lym/config/gate.yaml keys as the evidence and add an acceptance criterion for the chosen route.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:120 (D7 step 1)
    summary: D7 step 1 rests on the gate's existing diff route, which returns changed paths only, so the function text at the merge base it compares against has no named source, and a worktree with no parent branch is unhandled.
    fix: In D7 step 1, record that internal/gatecli's cardChanges and impactset.ChangedPaths return only paths, name the fabricengine read that returns a Go file's text at the merge base (research whether an existing call such as AnchoredFileAt serves the weft, or state that fabricengine gains one, marked (new)), and state what step 1 does when parentMergeBase reports no parent branch, with a matching acceptance criterion.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:95, 103, 111 (D5 exit count, D6 trigger and bound)
    summary: A package D5 skips as dirty or locked is reported as stale and unbuilt, and D6 restarts a refresh whenever packages were left unbuilt, so one dirty file in the prime spawns a refresh on every tick, contradicting D6's own bound.
    fix: In D5, report skipped packages apart from packages left unbuilt by the --max cap and state that skips do not count against --max; in D6, retrigger only on capped leftovers; rewrite the D6 acceptance criterion at line 188 so a refresh that only skipped packages does not retrigger.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:85, 90, 186-187 (D5 staleness and dirty skip)
    summary: D5 counts an untracked Go file as making a package stale but also skips any package with an untracked Go file, so such a package is never rebuilt and the two D5 acceptance criteria contradict each other.
    fix: In D5, decide whether an untracked Go file makes a package stale or makes it skipped, state that one rule, and rewrite the two D5 acceptance criteria at lines 186-187 to match it.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:88-97 (D5)
    summary: A Finalize landing fast-forwards the prime's checkout in place while lym gate testmap runs tests there, so a map can record one tree's commit and hashes over coverage from another, and D5 states no ordering against the landing.
    fix: In D5, state how a refresh stays consistent with a concurrent landing, for example that it records HEAD and the file hashes before each package and discards that package's map when HEAD or a hash differs after its tests, or that it takes the weft write lock fabricengine's merge takes; add an acceptance criterion for a HEAD change mid-package.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:92-97 (D5 bound)
    summary: lym gate testmap has no per-test timeout, so a hung test holds its gate slot without end and D6's no-refresh-in-flight rule then blocks every later refresh; the bound counts packages, not time.
    fix: In D5, give each test a timeout, reusing redundancy_test_timeout_sec or a new test_map key with a default, state that a timed-out test is left out of the map, and restate the bound in time as well as packages; add the case to the D5 acceptance criteria.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:137-138 (D7 bound)
    summary: The claim that the step flags falsely only when a map test lost coverage since the map was built, bounded by the age limit, is wrong, because the diff itself can make an unchanged map test stop reaching an unchanged block.
    fix: In D7's bound, replace the age-limit claim with the true one, that a map test counts as a cover whenever its own function text is unchanged, so an edit to a helper or a non-test callee in the diff can produce a false contained verdict, bounded only by review reading each finding against the covering tests.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:131-132 (D7 failure paths)
    summary: The never-changes-the-gate-result rule lists build failure, failing or timed-out test and missing map, but not an unreadable or corrupt map, a failed report write, an unreadable merge base or a failed lym build.
    fix: In D7, state the rule over every error the step can meet, for example that any error of the step becomes a report line and never an exit code, and extend the D7 acceptance criterion at line 195 to an unreadable map and a failed report write.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:118, 136 (D7 placement and cost)
    summary: The step runs again after carried tiers on an unchanged tree, and the budget does not count coverage builds, the lym build or a test started before the budget ran out, so the stated cost understates the worst case.
    fix: In D7, either skip the step when the existing report matches the tree by D8's match, or state the repeat cost as a bound; restate the cost bound to include builds and the last test's timeout past the budget.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:103-106 (D6 spawned process)
    summary: The refresh's in-flight state lives in the dobby's memory, so a dobby stop or restart leaves the child holding a slot and package locks with no owner record and no stated ender, and the new dobby starts a second refresh.
    fix: In D6, state that the child registers under runproc.HubOwner with a named role and what ends it on a dobby stop, or that a restarted dobby reads an in-flight record before starting another, and state the bound.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:121 (D7 step 2)
    summary: A //testtiming:keep line on a new test drops it from judgment, and D7 states no bound on that escape.
    fix: In D7, add to the bound that a kept new test is listed in the report as not judged with its keep reason, so Code-Review reads the reason in the diff, and that this is the escape's only check.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:121-124 (D7 steps 2 to 4)
    summary: Step 2 drops tests by D1's run-result rules (skipped, failed, no blocks) before step 4 runs them, so the drop order cannot be applied and such tests use cap slots.
    fix: In D7, keep only the static drops (tier, undeclared tier, keep, out-of-process scan) in step 2 and move the skipped, failed and no-blocks drops after step 4's runs.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:131, 143 (D7 red report, D8 match)
    summary: A red-gate report records none of the hashes D8's match reads, so whether Code-Burler-Review shows no findings or the no-current-report line is undefined.
    fix: In D8, state that a report written on a red tier never matches, so the marker fills the no-current-report line naming the red gate, and add that case to the D8 acceptance criterion.
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:129 (D7 step 6)
    summary: For a package the run edits, the changed-files line lists the diff's own files, so it cannot tell map drift from the run's own edits.
    fix: In D7 step 6, name only the files that differ from the map's hashes and are unchanged since the merge base, so the line shows drift on the parent alone.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:169-176 (Auto-mode assumptions, Open risks)
    summary: The assumption that the prime's worktree updates in place on a landing is a fact, not a pick, and stands without evidence while Open risks reads None.
    fix: Move the line from Auto-mode assumptions to Constraints, citing internal/fabricengine/merge.go's merge, which runs git merge --ff-only in the parent's weft checkout under the weft write lock, so the prime's HEAD moves in place on a landing.
  - id: F16
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:128 (D7 step 6)
    summary: Contained with the fewest covering tests claims a minimum cover, but the reused coveringTests picks the largest overlap first and computes no minimum.
    fix: In D7 step 6, write contained with its covering tests, largest overlap first.
  - id: F17
    severity: NIT
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:143 (D8 match)
    summary: D8's match ignores the map, so a map rebuilt between the gate run and the review leaves a matching report built against the old map, and the record states no bound for that.
    fix: In D8's rationale, add the bound that a map rebuilt since the report can make a contained verdict stale, and that review reading each finding bounds it.
---

### [BLOCKING] The `reports` step never reaches a hub that tunes its gate keys

**Location:** `_lym/discussion/decision-record.md:133`, D7.

**Issue:** D7 ships the step by listing `reports: [redundancy]` on four keys of `internal/gateslot/template.yaml`.
This hub's base `_bolt/_lym/config/gate.yaml:46-51` sets `webster`, `parent-align`, `darn-write` and `code-burler-fix` whole, with tiers `[0, 1, 2]` and no `reports`, and a hub base file's tuned unit wins over the template.
So on this hub the step never runs, and the goal's "stop piling up" is unmet.
No Decision says how a hub that tunes these keys gets the step.
Found by existing-paths and walkthrough and by the handler.
I raised it to BLOCKING because the feature D7 to D9 build does nothing on the hub it is for.

### [MEDIUM] D7 step 1's diff route gives paths, not base text

**Location:** `_lym/discussion/decision-record.md:120`, D7 step 1.

**Issue:** `cardChanges` returns `impactset.ChangedPaths(ctx, root, base)`, which lists paths (`internal/gatecli/scope.go:476-495`).
Telling which test functions differ from the merge base needs each file's text at that base, a git read at a revision that `PATTERN-fabric-boundary` keeps to `fabricengine`.
The record names no such read; the one file-at-revision call found, `fabricengine.AnchoredFileAt`, is anchor-relative, and the record does not say whether it serves the weft.
`parentMergeBase` also reports false for a worktree with no parent branch (`scope.go:668-677`), and D7 does not say what step 1 judges then.
This is a fact the design rests on, left for Plan.
Found by all three of walkthrough, bound-breaker and existing-paths, and by the handler.

### [MEDIUM] Dirty or locked skips retrigger a refresh on every tick

**Location:** `_lym/discussion/decision-record.md:95, 103, 111`, D5 and D6.

**Issue:** D5 skips a dirty or locked package and still counts it among stale packages left unbuilt.
D6 starts `lym gate testmap` on any tick where the last refresh left packages unbuilt.
One dirty file in the prime therefore makes the dobby start a refresh, taking a gate slot, on every tick.
That contradicts D6's bound that a dirty package waits for the next prime HEAD change, and the D6 acceptance criterion at line 188 encodes the loop.
Found by interactions, bound-breaker and walkthrough.

### [MEDIUM] Untracked files make a package both stale and always skipped

**Location:** `_lym/discussion/decision-record.md:85, 90, 186-187`, D5.

**Issue:** An untracked Go file marks the package stale (line 85) and always differs from HEAD, so the package is always skipped (line 90).
The criterion that the verb rebuilds packages with an untracked file cannot pass alongside the criterion that it skips one differing from HEAD.

### [MEDIUM] A landing can land under a running refresh

**Location:** `_lym/discussion/decision-record.md:88-97`, D5.

**Issue:** Finalize's merge runs `git merge --ff-only` in the parent's weft checkout under the weft write lock (`internal/fabricengine/merge.go:413-418`).
D5 checks for dirt once and takes only its per-package lock, so a landing into the prime mid-refresh changes files under running tests.
The map then records one tree's commit and hashes over coverage from another, and reads as fresh until the next HEAD change or the age limit.
Found by existing-paths and bound-breaker and by the handler.

### [MEDIUM] `lym gate testmap` has no per-test timeout

**Location:** `_lym/discussion/decision-record.md:92-97`, D5.

**Issue:** D7 gives each test `redundancy_test_timeout_sec`; D5 gives none.
A hung `tier1` test holds the slot without end, and D6's no-refresh-in-flight rule then blocks every later refresh.
D5's bound limits packages per call, not time.

### [MEDIUM] D7's false-flag bound is wrong

**Location:** `_lym/discussion/decision-record.md:137-138`, D7 bound.

**Issue:** A map test counts as a cover while its own function text is unchanged.
A diff that edits a non-test callee or a `_test.go` helper can stop `TestA` reaching block B, while B's key and `TestA`'s hash stay the same, so a new test covering B is reported contained.
The drift comes from the diff, not from time, so the age limit does not bound it.
Found by bound-breaker and by the handler.

### [LOW] Failure paths the never-changes-the-result rule leaves out

**Location:** `_lym/discussion/decision-record.md:131-132`, D7.

**Issue:** The list omits an unreadable or corrupt map, a failed report write, an unreadable merge base and a failed `lym` build, and no criterion reaches them.

### [LOW] Repeat runs and an understated budget

**Location:** `_lym/discussion/decision-record.md:118, 136`, D7.

**Issue:** The step runs after carried tiers too, so every call on an unchanged tree pays the budget again (`internal/gatecli/green.go`'s carry).
The budget stops starting tests but counts no coverage build, no `lym` build, and no test started just before it ran out.
Merges existing-paths F5 and bound-breaker finding 8.

### [LOW] The dobby's refresh child has no owner after a restart

**Location:** `_lym/discussion/decision-record.md:103-106`, D6.

**Issue:** In-flight state lives in the dobby's memory.
A stop or restart orphans the child with its slot and locks, names no ender, and lets the new dobby start a second refresh.
`PATTERN-child-processes` places a hub process under `runproc.HubOwner`.
Found by existing-paths and bound-breaker and by the handler.

### [LOW] The keep escape has no stated bound

**Location:** `_lym/discussion/decision-record.md:121`, D7 step 2.

**Issue:** The rubric's attack-surface rule asks each escape hatch to state what bounds it; D7 does not for `//testtiming:keep` on a new test.

### [LOW] Step 2 drops by results step 4 has not produced

**Location:** `_lym/discussion/decision-record.md:121-124`, D7.

**Issue:** Skipped, failed and no-blocks are known only after a run, yet step 2 drops them before the cap and the runs.

### [LOW] A red report against D8's match

**Location:** `_lym/discussion/decision-record.md:131, 143`, D7 and D8.

**Issue:** A red-gate report holds no hashes, so D8's match over it is undefined, and a red `code-burler-fix` attempt can overwrite a green `webster` report.

### [LOW] The changed-files line mixes the run's edits with drift

**Location:** `_lym/discussion/decision-record.md:129`, D7 step 6.

**Issue:** A package the run edits always differs from the map, so the line lists the run's own files and cannot warn about drift on the parent.

### [LOW] An unverified fact sits under Auto-mode assumptions

**Location:** `_lym/discussion/decision-record.md:169-176`.

**Issue:** That the prime's worktree updates in place on a landing is a fact D6 rests on, not an auto-pick.
It is true: `internal/fabricengine/merge.go` runs `git merge --ff-only` in the parent's weft checkout.
The record gives no evidence for it, and Open risks reads `None` beside it.
The other half of the focus question holds: the prime has its own dobby, which `internal/dobbyengine/doc.go` and `announce.go` name as "the prime's dobby".

### [NIT] Fewest covering tests is not what the engine computes

**Location:** `_lym/discussion/decision-record.md:128`, D7 step 6.

**Issue:** `coveringTests` in `cmd/testtiming/redundancy.go:186-235` takes the largest overlap first, which is no minimum.

### [NIT] D8 does not state the map-rebuild bound

**Location:** `_lym/discussion/decision-record.md:143`, D8.

**Issue:** A map rebuilt between the gate run and the review leaves a matching report computed against the old map.

## Focus verdicts

- D6 and the auto-mode assumption: both facts hold in code, but the record gives no evidence (F15). Leaving the fact to Plan is not the issue; the record states it without its source.
- D7 step 1 and D8: the route gives paths only and the record leaves the base-text read to Plan (F2).
  The `card` key's base is `parentMergeBase`, the merge base with the origin record's parent branch, which the record names correctly.
- D2 and D4: not a defect.
  `cmd/testtiming/redundancy.go:378-382` builds `-coverpkg` from every package, the package under test included, and Go's cover tool instruments each file the same whatever the set, so one module-wide profile holds the package's own blocks.
  D4's package-only binary yields the same block set for the package's files, and D3's content keys make positions irrelevant anyway.
- D5, D6 and D7 attack surface: the prime-worktree refusal and the `reports` list state their bounds; the keep escape (F11), the testmap time bound (F6), the dobby child (F10) and the failure paths (F8) do not.
- D3, D4, D7 and D8 fields: the function hash, block key and per-package file hashes agree across D3, D4, D7 step 6 and D8, so D8's match is computable, except over a red report (F13).
- Whole record: no text needs relocating to `support-log.md`; Open risks reads `None` beside an unverified assumption (F15).

## Rejected

- Fork-suggested severity MEDIUM for the hub-config finding: raised to BLOCKING, since the step would never run on this hub.
- No other fork finding was rejected; duplicates across lenses were merged into the findings above.
