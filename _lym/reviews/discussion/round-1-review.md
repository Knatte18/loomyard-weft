---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:141
    summary: D8 rests on the false premise that internal/agentconfig refuses no unknown role; Load refuses a skills or efforts key naming a role outside the roles lym starts, so a hub agents.yaml naming treadle-judge stops loading once the role is deleted, and D8's acceptance criterion fails as designed.
    fix: In D8, replace the sentence about agentconfig with the observed behavior and its pin (internal/agentconfig/config.go Load and skillLists; config_test.go rows UnknownRole and PairWithUnknownRole), and decide that a configreg.Reinterpretation on the agents module strips the treadle-judge and treadle-targeting keys from skills and from role/model efforts keys, applied by hubreconcile per PATTERN-config-layers. Rewrite the D8 acceptance criterion to - a hub agents.yaml naming treadle-judge is rewritten by the reinterpretation and then loads (D8). In support-log.md, correct the Turn 3 bound-breaker line that records the false fix.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:74
    summary: The live gate runs lym gate test --tier 2 and --tier 2 --copies over a package list, naming a tier instead of a gate key, against PATTERN-test-speed's declared text (via lym gate test by gate key) and its background rule that a site names a gate key, never a tier list.
    fix: In D4, have the live gate run two hub gate.yaml keys shipped the way code-burler-fix is, for example harden-live (tier 2, the serial run) and harden-live-copies (tier 2, the concurrent run), through lym gate test --gate <key> with the packages as targets; make --copies go with --gate and state that bound. Replace --tier 2 in D4 steps 1 and 2, in the --copies bound and in the D4 acceptance criteria with the key form, and keep step 3's rule that no key names tier 3.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:98
    summary: D5's reaps kill the sandbox hubs' processes found by a /proc scan, but PATTERN-child-processes ends a run's processes only through runproc's End, EndSparing, EndRole or EndSubject, with EndProcess reserved for lym tender reap's orphans; D5 names no runproc route and declares no PATTERN change.
    fix: In D5, have the reaps first end each sandbox hub's recorded processes through runproc (End over that hub's owners), and kill by the /proc scan only what no record names, stating that this second route is a new orphan reaper. Add the record's PATTERN changes section with a changes bullet for PATTERN-child-processes that admits the Hardener sandbox reaper beside lym tender reap as a user of EndProcess, bounded to processes under the sandbox root; add a matching acceptance criterion.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:80
    summary: --copies cannot run concurrent copies from inside the live gate, because a lym gate test nested in an enclosing gate's held slot acquires no slot of its own; the gate's one slot gives one copy every round, so the concurrent amplifier never runs and the shortfall bound describes a partial grant the pool never makes.
    fix: In D4, state the observed slot rules with their pin (internal/gateslot/doc.go - a nested gate run runs inside its parent's slot without acquiring; a multi-slot acquire waits for every slot and never grants part). Then decide how the copies get n slots, for example the live gate releases its own hold before the copies call, or the live gate acquires n slots itself and the nested call runs n copies inside that inherited hold, and rewrite the shortfall bound and the --copies acceptance criterion to the chosen mechanism.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:76
    summary: D4 calls the step-1 tier-2 run serial, crucible's merge bar, but the hub's tier 2 runs go test with -p=4 from its parallel setting and tests call t.Parallel per PATTERN-test-speed, so the run is concurrent; the claim is unpinned and false.
    fix: In D4 step 1, either drop serially and restate the merge bar as one green tier-2 run, or have the serial key of F2 pin -p=1 and -parallel=1 through its args or env, and cite the pin (the hub gate.yaml tier2 entry, parallel 4 and GOFLAGS -p={parallel}).
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:120
    summary: CONVERGED needs a reviewer model different from the previous round's, but a one-entry or adjacent-duplicate reviewer list set through hardener.yaml or run_settings, and every round a parent continue runs past max_bounces, where the clamp repeats the last expanded entry, can never meet it, so such a campaign only ends capped.
    fix: In D3, expand the reviewer and fixer lists cyclically to the Harden perch's hard cap, not to max_bounces, and state that hard cap in D9. In D2 or D9, have Preflight refuse a reviewer list whose cyclic expansion places the same model in two adjacent rounds, with the run_settings write as its way forward. Extend the rotation criterion to a round past max_bounces, and add a criterion for the Preflight refusal.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:122
    summary: The live gate re-prompts the fixer on a red tier-2 run, but D6 has the fixer defer a finding whose fix lies outside the module; when such a deferred cause keeps the gate red, the record states neither the live gate's attempt count nor what the round does when they run out.
    fix: In D4, give the live gate an attempt count from hardener.yaml (D9 key live_attempts) and state its exhaustion outcome, for example the row ends the round with the live report marked red and the gate failures recorded as deferred, so the Bouncer's carry-over and CONVERGED read it, instead of halting the run. Add an acceptance criterion for that outcome.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:98
    summary: Four reaps per round, at the start and end of both the review and fix rows, write the round's stray report, but D6 names one per-round file with no append or per-row rule and D7 archives it at the fix row's end without ordering the archive against that row's end-of-row reap, so the review row's strays or the fix row's last reap can miss the facts and the archive.
    fix: In D5, name one stray report per row and reap, or one per round that each reap appends to with the row and reap point on every entry. In D7, state that the fix row's end-of-row reap runs before the archive copy. Add to the stray-report criterion that a stray left by the review row appears in round-<n>'s archived report.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:166
    summary: D10's bound names which recipes keep today's check but not what the loosened order clause now lets through - a HardenFix row whose segment's Bouncer lies after Record-Write but which no on_stuck edge of that Bouncer reaches passes the contract and builds nothing, and a segmented build row whose segment holds no Bouncer has no stated outcome.
    fix: In D10, add to the rule that a segmented build row must be reached from its segment's Bouncer by the ring's edges, and that a segmented build row whose segment holds no Bouncer is refused with its way forward; state these as the clause's bound. Add both refusals to the D10 acceptance criterion.
  - id: F10
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:207
    summary: The Open risks treadle question stands - D8's rationale cites the 2026-10-16 decision as keeping treadle's stencils only until Hardener is built, but the entry says they are kept for Hardener and re-derived from the Bouncer judge when it is built, and the 2026-11-18 operator decision names treadle among the modules to build on; neither settles whether treadle may be deleted.
    fix: Raise to the run's parent - may this run delete internal/treadleengine, its stencils and roles (D8), or must they stay? Once answered, the writer drops the Open risks line and restates D8's rationale to quote the board entry's note as written.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:101
    summary: D5's reap bounds miss three cases - a reed server whose hub directory is already gone, since the socket key derives only from an existing hub path; a hub or pair a round agent makes outside the root; and a root still holding hub repositories at teardown, which fabric's leftover cleaners refuse because they hold a .git.
    fix: In D5's Bounds, have the reap also list live reed servers whose server-name readable half or recorded hub path lies under the root, so a removed hub's server is found; state that a hub outside the root is outside the reap and the rubric forbids making one; and have Finalize and the capped or circling halt path run the reap before teardown, citing internal/fabricengine/destroy.go's .git refusals as the reason.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:63
    summary: The Harden perch reads its round cap from hardener.yaml in the full wiring, but the lightweight status and routing projection rebuilds segment budgets from loom.yaml and tapestry.yaml alone through loomSegmentBudgets, so it reports no cap or loom's for Harden-Review.
    fix: In D3, state that the segment-budget projection, the lightweight reader in internal/loomcli/arm.go included, takes Harden-Review's cap from hardener.yaml, and add a criterion that lym loom status on a hardener run reports max_bounces as the Harden-Review budget.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:61
    summary: D3 has HardenFix's verify gate run the hub key harden-burler-fix, while D1 and D9 make tree_scope the run's verify scope, and darn's verify gate runs the tree-scoped command or the code-burler-fix key; the record leaves open which one HardenFix's verify runs.
    fix: In D3, state which source the verify gate takes for a hardener run - the harden-burler-fix key alone, or tree_scope's command with harden-burler-fix as the fallback as darn does - and align D9's tree_scope description to it.
  - id: F14
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:68
    summary: Two code facts carry no pin - D3's rationale that burler's per-round lists clamp to their last entry, and D1's rationale that batten refuses a child recipe missing from its list.
    fix: Cite the source beside each claim - internal/burlerengine/models.go RoundModels.Pick for the clamp, and the batten function that refuses an unlisted child recipe, by path, for D1.
---

### [BLOCKING] D8's agentconfig premise is false

**Location:** `_lym/discussion/decision-record.md:141`
**Issue:** `agentconfig.Load` refuses a role key outside the roles lym starts (`internal/agentconfig/config.go`, `skillLists`; `config_test.go` rows `UnknownRole` and `PairWithUnknownRole`).
Deleting the treadle roles therefore breaks any hub whose `agents.yaml` names them, and that meaning change needs the reinterpretation PATTERN-config-layers requires.
Every lens found this.

### [BLOCKING] The live gate names a tier instead of a gate key

**Location:** `_lym/discussion/decision-record.md:74`
**Issue:** PATTERN-test-speed's declared text runs tests "via `lym gate test` by gate key", and its background says a site names a gate key, never a tier list.
D4 runs `lym gate test --tier 2`, with and without `--copies`, while D3 already gives the verify gate a key.

### [BLOCKING] The sandbox reaper ends processes outside runproc

**Location:** `_lym/discussion/decision-record.md:98`
**Issue:** PATTERN-child-processes' Run process owner section ends a run's processes only through runproc and keeps `EndProcess` for `lym tender reap`'s orphans; it calls another route a review finding.
The sandbox hubs' processes are runs of those hubs, started by the `.dev-bin` lym, and D5 kills them by a `/proc` scan with no PATTERN change declared.

### [MEDIUM] --copies cannot get slots inside the live gate

**Location:** `_lym/discussion/decision-record.md:80`
**Issue:** `internal/gateslot/doc.go` says a nested gate run runs inside its parent's slot without acquiring, and a multi-slot acquire waits for every slot rather than granting part of them.
The copies call inherits the live gate's hold, so it runs one copy every round.

### [MEDIUM] The step-1 tier-2 run is not serial

**Location:** `_lym/discussion/decision-record.md:76`
**Issue:** The hub's `gate.yaml` tier 2 has `parallel: 4` and `GOFLAGS -p={parallel}`, and tests call `t.Parallel` by rule.
The "serial" merge bar D4 cites from crucible does not hold.

### [MEDIUM] Model rotation can make CONVERGED unreachable

**Location:** `_lym/discussion/decision-record.md:120`
**Issue:** The clamp in `burlerengine.RoundModels.Pick` repeats the last entry past the expanded list, and D9 lets `run_settings` set a list with one entry.
Either case gives two adjacent rounds the same reviewer, which the different-model rule refuses.

### [MEDIUM] A deferred cause of a red live gate has no outcome

**Location:** `_lym/discussion/decision-record.md:122`
**Issue:** The live gate re-prompts the fixer, and the fixer is told to defer a fix outside the module.
The record states no attempt count for the live gate and no outcome once its attempts run out.

### [MEDIUM] Stray-report writers and the archive order disagree

**Location:** `_lym/discussion/decision-record.md:98`
**Issue:** D5 has four reaps per round, D6 lists one file, and D7 does not order its copy against the fix row's end-of-row reap.
The review row's strays and the last reap's record can miss both the facts and the archive.

### [MEDIUM] D10's loosened order clause has no stated reach

**Location:** `_lym/discussion/decision-record.md:166`
**Issue:** `checkLoomContract` compares only path indices.
With the segment's Bouncer standing in for the build row, a fix row no edge reaches passes, and a segment with no Bouncer has no stated outcome.

### [MEDIUM] Treadle deletion needs the parent

**Location:** `_lym/discussion/decision-record.md:207`
**Issue:** The entry's 2026-10-16 note keeps the treadle stencils for Hardener, to be re-derived from the Bouncer judge when Hardener is built.
The 2026-11-18 operator decision lists treadle among the modules to build on.
D8's rationale reads the older note as "kept only until Hardener is built", which the note does not say, so the rationale cannot answer the Open risks question, and the line stays for the parent.

### [LOW] D5's reap bounds miss three cases

**Location:** `_lym/discussion/decision-record.md:101`
**Issue:** A server whose hub is already gone yields no derivable key (`internal/reedengine/server.go` `ServerName`).
A hub made outside the root is outside every reap.
Fabric's leftover cleaners refuse a directory holding a `.git` (`internal/fabricengine/destroy.go`).

### [LOW] The lightweight budget reader ignores hardener.yaml

**Location:** `_lym/discussion/decision-record.md:63`
**Issue:** `loomCLI.reviewBudget` rebuilds budgets through `loomSegmentBudgets` from `loom.yaml` and `tapestry.yaml` alone.

### [LOW] HardenFix's verify source is ambiguous

**Location:** `_lym/discussion/decision-record.md:61`
**Issue:** D3 names a gate key, while D1 and D9 name `tree_scope` as the verify scope, and darn's verify uses both.

### [LOW] Two code facts are unpinned

**Location:** `_lym/discussion/decision-record.md:68`
**Issue:** The burler clamp holds (`internal/burlerengine/models.go`), but the record cites nothing for it, and nothing for batten's refusal of an unlisted child recipe.
D8's agentconfig claim is the third fact the focus named; it is false and is F1.

## Rejected

- Walkthrough finding 5, that a clean review may skip the live gate: burler runs the round's gates first and spawns the fixer only when they fail (`internal/burlerengine/prompt.go` `fixWork`), so a clean round still runs `live`.
- Interactions finding 7, that the rotation criterion covers rounds 1 to 7 only: merged into F6, whose fix extends the criterion.
- Focus: whether the verbs' refusal outside a hardener worktree covers the Go-side reaps. It need not: the reaps run only inside Harden rows of a hardener run, and derive the root from that run's worktree.
- Focus: deferred research on how `run_settings` keys reach `hardener.yaml`. `configreg.PrepareRunSettings` turns any registered module's object into its local layer, so D9's registered unit needs no further fact.
- Focus: completeness-before-leanness over the support log. No relocation finding is raised, so no requirement needs extracting from it.
