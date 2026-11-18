---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:79
    summary: D3's purpose fails for the darn-write and code-burler-fix keys, because the writer never sees a passing post-session verify's stderr, so the reach note reaches no writer before review
    fix: In D3, name the channel that carries the note to the writer of a gate run through verifytree.Verify. For example, the loomshed verify gate turns a pass whose reach report lists unreached changed functions into one non-failing finding turn for the writer, bounded to one such turn per HEAD tree. Or, if no such channel is wanted, drop the claim and say the darn-write and code-burler-fix note serves the code review alone. Either way, rewrite the D3 rationale sentence and add an acceptance criterion for the chosen path.
    origin: lens:walkthrough
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:128
    summary: Triage lines count toward the failure report's 80-line cap, so they can cut a report that would otherwise fit; gateslot.FailedTests returns ok=false on a cut report, which breaks D7's unchanged-parse claim and its acceptance criterion
    fix: In D7, replace the bullet "It counts toward the report's line cap like the test's other output lines" with a rule that a triage line never causes a cut. A triage line is added only while the report's line budget still holds it, and is dropped before any test output is cut. Extend the D7 parse criterion to a report at the cap, where the report with triage lines parses to the same failed tests as without them.
    origin: lens:bound-breaker
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:70
    summary: D3 refreshes a package whose own files are clean, but the --map write also refuses when a linked package has a modified or untracked file, so such a refresh spends reach_budget_min and writes no map, and the note gives no reason or the wrong one
    fix: In D3's second bullet, refresh a package only when neither it nor any in-module package its test binary links has a modified or untracked file. The note names the dirty linked package as the reason. Cite pattern/PATTERN-test-speed.md, the sentence that a map is written only when no file the package or a linked package owns is modified or untracked. Update the matching D3 acceptance criterion and the Open risks batch bullet.
    origin: lens:existing-paths
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:92
    summary: D3 refreshes only the packages that hold changed functions, so a new function reached only from another package's tests is absent from every map and D4 lists it as reached by no test, a false unreached entry the writer is meant to act on
    fix: In D4, state the bound. A changed function that is new to its package's map, and whose package is linked by other packages' tests, may be reached by tests whose maps predate it. The note marks such a function as new since the cross-package maps, never as plainly unreached. Add that caveat to the D4 acceptance criterion beside the tier 2 and spawn caveat.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:124
    summary: D7 defines no case for a call with no parent branch or an unreadable merge base, where D2 yields no change set, so every mapped failing test would get reaches none of this run's changes and the D8 hint, a false signal that the failure is not the run's
    fix: In D7, add a bullet. When the call has no parent branch, or its merge base cannot be read, the triage lines say that no change set exists, never "reaches none of this run's changes", and carry no D8 command. Add a D7 acceptance criterion for that case.
    origin: lens:interactions
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lym/discussion/decision-record.md:229
    summary: The board entry orders the mutation trial first and the step only if the trial pays; the record builds the planner, step and verb in this run before any trial, a departure from the entry that only the parent can settle
    fix: Raise to the parent. Either this run builds D9 to D11 now with the step unlisted, as the record has it, or the parent runs the trial first as its own darn and this run drops D9 to D11. Record the answer with lym loom decision add; the writer then drops or keeps the Open risks bullet accordingly.
    origin: handler
  - id: F7
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:159
    summary: In the prime D3 refreshes nothing and the prime keeps no anchor maps, so D9 finds no fresh map and lym gate mutate or the mutation step there reports every package not judged, a case no decision states
    fix: In D9's first bullet, either count a hub map whose file hashes match the worktree as fresh in the prime, or state that a call in the prime judges nothing, with a note as D3 gives. Say the same in D11, and add the case to the D9 acceptance criterion.
    origin: lens:walkthrough
  - id: F8
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:43
    summary: The index stamps each listed map file, so a map file the index never listed, such as one written by a --map run that ended before its index write, leaves every stamp matching and readers miss that package
    fix: In D1's index bullet, stamp the set of map files in the directory as well as each file's size and modification time, so an added or removed map file counts as a stamp difference. Extend the D1 stale-index acceptance criterion to an added map file.
    origin: lens:bound-breaker
  - id: F9
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:58
    summary: A function key carries its file, so a function moved verbatim to another file of its package counts as new under D2, and several init functions of one file share a key, so their text comparison is ambiguous
    fix: In D2, match a function by package and name across the package's files, so a verbatim move between files is unchanged. Compare init functions of one file in order. Add the moved-across-files case to the D2 acceptance criterion.
    origin: lens:bound-breaker
  - id: F10
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:101
    summary: D5 says the reach report is matched as the redundancy report is, but the redundancy note never matches a report written on a red tier, while D5's match on functions and file hashes lets a red-tier reach report that judged nothing stand for HEAD
    fix: In D5, add that a report written on a red tier, outside a hub or under another scoper never matches. The note then reads the no-current-report line naming the reason. Add the red-tier case to the D5 acceptance criterion.
    origin: lens:bound-breaker
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:116
    summary: D6 refuses a function with --test, or neither, but says nothing of --since or --branch given with the function form
    fix: In D6's refusal sentence, also refuse --since or --branch without --test, naming the D8 form. Add that case to the D6 acceptance criterion.
    origin: lens:interactions
  - id: F12
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:142
    summary: D8 reuses D2 between a commit and its first parent, but D2 is defined as base contents against the worktree's files, so the walk hands D2 an input its contract does not take
    fix: In D2, state that the finder compares two given sets of file contents, the base and the head, and that the reach step hands it the worktree's files as the head. D8 then hands it each commit and its first parent through fabric.
    origin: lens:walkthrough
  - id: F13
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:236
    summary: The Open risks batch bullet says a tapestry writer's batch gate runs on uncommitted edits, but the weaver's batch gate runs through verifytree.Verify, which refuses a dirty tree; only a hand-run gate call meets uncommitted edits
    fix: Narrow the bullet to a writer's hand-run lym gate test --gate batch call on uncommitted edits. In the D3 rationale, name the weaver's batch gate beside Darn-Write and Code-Burler-Fix as running through verifytree.Verify.
    origin: lens:bound-breaker
  - id: F14
    severity: NIT
    class: design
    location: _lym/discussion/decision-record.md:149
    summary: A failure report line is cut at 300 bytes, and a triage line naming long function keys plus the trailing D8 command can pass that, cutting off the command
    fix: In D7, place the D8 command on its own four-space line under the triage line, so the line cap never cuts it.
    origin: lens:bound-breaker
  - id: F15
    severity: NIT
    class: consistency
    location: _lym/discussion/decision-record.md:225
    summary: The auto-mode assumption that the mutation trial runs outside this run restates D11 and the Open risk
    fix: Delete the bullet "The mutation trial runs outside this run (D11)." from Auto-mode assumptions.
    origin: handler
---

### [BLOCKING] The reach note never reaches the darn-write or code-burler-fix writer

**Location:** `_lym/discussion/decision-record.md:79`

**Issue:** D3's rationale is that the writer reads the note from its own gate before review, and it names the Darn-Write and Code-Burler-Fix gates.
Those gates run after the writer's turn, through `verifytree.Verify` in `internal/loomshed/verifygate.go`.
A pass returns `GateResult{Passed: true}` with no findings, and a skipped tree runs nothing.
The darn writer's gated stencil, `darn-fragment-make-change-gated.md`, tells it to run no gate itself.
So the note stays in a verify log that no writer reads, and the operator's ask the record cites goes unmet for both keys.

### [MEDIUM] Triage lines can cut a failure report that would otherwise parse

**Location:** `_lym/discussion/decision-record.md:128`

**Issue:** `failureReportLines` is 80 across the whole report (`internal/gatecli/failurereport.go`), and `renderFailureReport` ends an overrun with a `... ` line.
`gateslot.FailedTests` returns `ok=false` on that line.
Both `internal/loomshed/verifygate.go` (`namedFailures`) and `internal/landingshed/verifygate.go` then fall back from running the failed tests alone to the whole command.
All four lenses found this.

### [MEDIUM] D3 refreshes packages the measurement will refuse

**Location:** `_lym/discussion/decision-record.md:70`

**Issue:** The `--map` write also checks every linked package's files (`internal/gatecli/testmap.go`, `checkClean`; `pattern/PATTERN-test-speed.md`).
D3 checks only the package's own files, so it spends budget on refreshes that write nothing, and the note gives no reason or the wrong one.

### [MEDIUM] New functions reached from other packages show as unreached

**Location:** `_lym/discussion/decision-record.md:92`

**Issue:** Cross-package reach comes from other packages' maps, which D3 never refreshes.
A new function tested only from another package is absent from every map.
D4 then lists it as unreached, the very list the writer is meant to act on.
D7 states this bound for triage; D3 and D4 do not.

### [MEDIUM] Triage with no change set claims the failure is not the run's

**Location:** `_lym/discussion/decision-record.md:124`

**Issue:** With no parent branch, `parentMergeBase` returns found=false (`internal/gatecli/scope.go`).
D7 would then mark every mapped failing test as reaching none of the run's changes and append the D8 hint, which steers the fixer wrongly.

### [MEDIUM] Trial order is the parent's call

**Location:** `_lym/discussion/decision-record.md:229`

**Issue:** The board entry `testmap-uses-design` puts the mutation trial first and builds the step only if the trial pays.
The record builds D9 to D11 now with the step unlisted.
The writer raised this correctly under Open risks.
It departs from the entry's stated order and scope, so only the parent can settle it.

### [LOW] The prime judges nothing in the mutation check

**Location:** `_lym/discussion/decision-record.md:159`

**Issue:** D9's freshness test admits only refreshed or matching worktree maps, and the prime has neither, so every package comes back not judged with no stated reason.

### [LOW] The index stamp misses an added map file

**Location:** `_lym/discussion/decision-record.md:43`

**Issue:** The stamps cover the files the index lists.
A map written after the index without an index rewrite, as when a moved HEAD ends the `--map` walk, leaves every stamp matching.

### [LOW] Function identity includes the file

**Location:** `_lym/discussion/decision-record.md:58`

**Issue:** `testcover.FuncKey` carries the file, so D2 calls a verbatim move between files a new function.
Several `init` functions in one file share a key.

### [LOW] A red-tier reach report can stand for HEAD

**Location:** `_lym/discussion/decision-record.md:101`

**Issue:** The redundancy note never matches a red-tier report (`pattern/PATTERN-test-speed.md`).
D5 claims the same fill but matches only on functions and hashes.

### [LOW] D6 says nothing of --since and --branch with the function form

**Location:** `_lym/discussion/decision-record.md:116`

**Issue:** D6 refuses only the function-and-`--test` combination and the empty call.

### [LOW] D8 hands D2 two commits, which D2's contract does not take

**Location:** `_lym/discussion/decision-record.md:142`

**Issue:** D2 is defined as base contents against the worktree's files; D8 needs a commit against its first parent.

### [LOW] The batch Open risk overstates the dirty case

**Location:** `_lym/discussion/decision-record.md:236`

**Issue:** The weaver's batch gate calls `verifytree.Verify` (`internal/weaverengine/strand.go`), which refuses a dirty tree.
Only a hand-run gate call meets uncommitted edits.

### [NIT] The D8 command can be cut from its triage line

**Location:** `_lym/discussion/decision-record.md:149`

**Issue:** `failureReportLineBytes` is 300, and five qualified function keys plus the command can exceed it.

### [NIT] An auto-mode assumption repeats D11

**Location:** `_lym/discussion/decision-record.md:225`

**Issue:** The bullet repeats what D11 and the Open risk already state.

## Rejected

- bound-breaker F9, D3's slot bound under `slots: all`: the `--map` run and the redundancy step already take the slot ask the hub configures for its tiers, and Bolt sets none; the reach step adds no new slot behaviour for D3 to bound.
- bound-breaker F1 at BLOCKING: kept as F2 at MEDIUM, since a cut report degrades the rerun to the whole command and loses no failure.

## Focus checks settled without a finding

- D7 and D3 code claims hold, each cited by its symbol.
  `gateslot.FailedTests` reads a four-space line as the test's own output, and `fixAndRerun` copies the first run's stderr as the card fixer's report.
  `verifytree.Verify` returns `StatusDirty` before running anything.
  The `--map` write refuses a dirty package, though the claim is incomplete (F3).
- D12's config fill holds: `internal/gateslot/doc.go` states it, and the hub's base `gate.yaml` tunes the affected keys without `reports`.
- The 56 MB, 153-function, four-flaky and 85 percent figures trace to the study report.
- D9's facts hold.
  `testcover.ProfileBlock` carries start and end lines, and `testcover.Mutant` carries `File` and `Line` for a per-line filter.
  The existing mutation runner in `internal/gatecli/redundantmutants.go` runs one test per job.
- The board note in D11 stays a note.
  No Decision edits this task's own entry or files or promotes a task.
- Relocation: no Decision rationale carries deliberation that belongs in the support log beyond F15.
