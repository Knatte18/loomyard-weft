---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:216
    summary: D9 lets a task's run_settings override every hardener.yaml key, but run_settings reach a run only as the pair's local layer, which holds only CategoryNone keys; auto_accept, the fans and tree_scope are guard keys by the repo's own precedent, packages and llm_driving narrow or switch off live-gate checks, the record classifies no key, and auto_accept is an unbounded escape hatch that lets a campaign whose live gate is red land without the parent decision D6 promises.
    fix: "In D9, replace \"A task's run_settings may override any of them.\" with a classification of every hardener.yaml key in the unit's ConfigSpec per PATTERN-config-layers, citing the precedents (internal/reviewbudget/reviewbudget.go Categories, auto_accept Guard; internal/loomengine/config.go ConfigSpec, fixer and split fans Guard; internal/darnengine/config.go, tree_scope Guard and set by no run_settings; internal/configengine/doc.go, the local layer holds only CategoryNone keys). Classify auto_accept, reviewer_fan, fixer_fan and tree_scope as Guard, hub-file only. Classify packages and llm_driving as CategoryNone and state each one's bound in D2: packages narrows only the live gate, never the verify gate, and Preflight refuses an empty or missing package; llm_driving true removes only the concurrent copies, whose failures never fail the gate (D4). Name the keys run_settings may set. In D6, bound auto_accept: a Harden-Bouncer auto-accept never accepts a round whose live report is red, which then escalates to the parent as a capped campaign does; or Preflight refuses a hardener run whose auto_accept is true, naming the hardener.yaml write as its way forward. Add acceptance criteria: a task run_settings setting a Guard hardener.yaml key is refused at seed, packages and llm_driving set through run_settings load, and the chosen auto_accept bound holds on a red live report."
  - id: F16
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:90
    summary: lym's hub sets harden-live and harden-live-copies to [2], and a tier's {test_targets} is narrowed to the target packages that hold a test of that tier (PATTERN-test-speed, Four tiers), so a packages list with no tier-2 test starts no test binary in either live run; the live gate is then green every round and CONVERGED's green live gate, D4's independent verification, rests on no run.
    fix: "In D2, add a Preflight refusal of a hardener run whose packages hold no test of a tier the hub's harden-live key names, read the way lym gate test narrows {test_targets} (from each test's tiergate.Require), naming as ways forward the run_settings write of packages and the lym config gate --set write of harden-live. In D4, have the live report record the test binaries each live run started, and have a run that started none read as red, so the D6 convergence rule never reads it as green. Add acceptance criteria: Preflight refuses packages holding no test of the live key's tiers, and a live run that starts no test binary is reported red."
  - id: F17
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:200
    summary: The operator entry Delete treadle when Hardener is built (F10) settles the treadle question, yet D8's rationale still says the 2026-10-16 decision kept the stencils only until Hardener is built, Open risks (line 286) still puts the question to the parent, and support-log Q13 still records it as open; the stale Open risks line re-raises a settled parent question every round, and the arbiter entry's replacement text (an open parent question, stencils re-derived) contradicts the later operator entry.
    fix: "In decision-record.md, delete the Open risks bullet that opens Question for the parent: the entry's operator note lists treadle. In D8's rationale, replace the clause the operator's 2026-10-16 decision kept its stencils only until Hardener is built. with: the board entry records an operator decision of 2026-10-16 that kept treadle's judge stencils for Hardener, unmaintained, to be re-derived from the Bouncer judge when Hardener is built; the operator's decision of 2026-11-18, recorded under Added after Discussion, replaces it and deletes treadle without re-deriving the stencils. Leave both Added after Discussion entries untouched. In support-log.md, change Q13 from open to resolved, citing the operator's 2026-11-18 Added after Discussion entry, and drop the parent may overrule clause."
  - id: F18
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    carries: F4
    location: _lym/discussion/decision-record.md:107
    summary: The --copies fix says the call acquires n slots and runs n copies inside them, but lym gate test takes slots per tier run through acquireSlot and pool.AcquireSlots and skips that acquire only when LYM_GATE_SLOT names a slot an enclosing process holds; with n equal to the pool (3 on lym's hub), the call holds every slot and its own tier run then waits forever on a fourth. Running each package's compiled test binary also bypasses the key's tier command, go-json counter and timeout.
    fix: "In D4's --copies paragraph, state the observed slot rules with their pins (internal/gatecli/run.go: acquireSlot takes a tier's slots through gateslot.Pool.AcquireSlots, and the call runs inside an enclosing slot only when LYM_GATE_SLOT names a held one, the exe.inherited path). Then decide that a --copies call's n-slot lease replaces its tier runs' own acquires, so its tier run holds no slot beyond the lease, and that the n copies are n concurrent runs of the key's rendered tier command over the same targets, each under the tier's counter, timeout, CPU watchdog and memory guard, not raw compiled binaries. Add to the bound that a --copies call holding every slot drains the hub's pool for the length of its call. Rewrite the --copies acceptance criterion to assert that a call with n equal to the pool's slot count completes and runs n copies."
  - id: F19
    severity: LOW
    class: design
    origin: lens:bound-breaker
    carries: F11
    location: _lym/discussion/decision-record.md:142
    summary: The F11 fix reaps the root only at Finalize and at a capped or circling halt, but an operator's lym fabric remove and batten's Worktree-Teardown end a pair through internal/pairteardown with no reap; the sandbox hubs' reed servers and dobbies are recorded in each sandbox hub's own runproc directory, not under the pair's slug, so they survive, and the root's nested .git keeps ignoredTaskPathDeletable refusing the leftover path.
    fix: "In D5's teardown bound, have internal/pairteardown run the sandbox reap for a pair whose run's recipe is hardener before it removes the worktree, and name every teardown route that reaches it: Finalize, the capped or circling halt, lym fabric remove and Worktree-Teardown. If pairteardown must not learn of Hardener, instead state as the bound that such a removal leaves the root's servers and repositories, and have the refusal or report it meets name lym harden sandbox clear, run in the pair's worktree, as the way forward. Extend the Finalize reap criterion to the chosen route."
prior:
  - id: F1
    status: resolved
    answer: D8 now cites agentconfig.Load and skillLists with config_test.go rows UnknownRole and PairWithUnknownRole, ships a configreg.Reinterpretation on the agents module applied by hubreconcile, and the criterion reads rewritten by the reinterpretation and then loads; the support-log line is corrected.
  - id: F2
    status: resolved
    answer: D4 runs the hub gate keys harden-live and harden-live-copies through lym gate test --gate with packages as targets and names no tier list.
  - id: F3
    status: resolved
    answer: D5 ends recorded processes through runproc.End and limits the /proc kill to processes no record names, and the PATTERN changes section declares the PATTERN-child-processes change in the bounded text the declared change carries.
  - id: F4
    status: open
    answer: The fix chose an n-slot hold for the call, but the call's own tier run still acquires its tier's slots; carried as F18.
  - id: F5
    status: resolved
    answer: D4 step 1 now calls harden-live the merge-bar run, and the serial claim is gone.
  - id: F6
    status: resolved
    answer: D3 expands the lists cyclically to hard_max_bounces, D2's Preflight refuses one-entry and adjacent-duplicate lists including a first-last match, and the criteria cover rounds past max_bounces.
  - id: F7
    status: resolved
    answer: D4 adds live_attempts from hardener.yaml and ends the row red with deferred failures; D6 and the criteria read the outcome.
  - id: F8
    status: resolved
    answer: D5 appends every reap to one per-round stray report with row and reap point, D7 copies after the end-of-row reap, and a criterion covers a review-row stray in the archive.
  - id: F9
    status: resolved
    answer: D10 refuses a segmented build row no Bouncer edge reaches and one whose segment holds no Bouncer, each with its way forward, and the first criterion names both.
  - id: F10
    status: resolved
    answer: "Settled by the added entry Delete treadle when Hardener is built (F10). The record's stale Open risks line and D8 rationale are raised as F17."
  - id: F11
    status: open
    answer: The hub list and the Finalize and halt reaps cover a gone hub's server and the two named teardowns, but pairteardown's operator removal runs no reap; carried as F19.
  - id: F12
    status: resolved
    answer: D3 takes Harden-Review's cap from hardener.yaml in the projection, loomCLI.reviewBudget included, and a criterion checks lym loom status.
  - id: F13
    status: resolved
    answer: D3 states that HardenFix's verify runs harden-burler-fix, over the touched trees' targets for a diff wholly under hardener.yaml's tree_scope, as darnTreeScope does.
  - id: F14
    status: resolved
    answer: D3 cites internal/burlerengine/models.go pickRound behind RoundModels.Pick, and Constraints cites internal/battencli/wire.go childRecipeSupported.
---

### [BLOCKING] hardener.yaml keys are unclassified and D9's run_settings override breaks the local-layer rule

**Location:** `_lym/discussion/decision-record.md:216` (D9), with D2 at line 49 and D6 at line 171.

**Issue:** Run settings reach a run only through the pair's local layer.
`configreg.PrepareRunSettings` runs `checkLocalDocument`, and `internal/configengine/doc.go:103` lets that layer hold only `CategoryNone` keys.
The repo's own precedent classes several D9 keys as Guard:
- `<perch>_auto_accept` (`internal/reviewbudget/reviewbudget.go:42`)
- the fixer and split fans (`internal/loomengine/config.go:649-656`)
- darn's `tree_scope`, "closed to the local layer; no task run_settings sets it" (`internal/darnengine/config.go:59`)

A task override of any of these is therefore refused at seed.
D2's charter keys meet the same "what a key's value can do to a guard" test (`pattern/PATTERN-config-layers.md`, Categories), yet the record classifies none of them.
Every lens raised this, and so did the handler.

`auto_accept` is also an escape hatch the record introduces with no bound.
With it set, a spent budget's CONTINUE or CIRCLING round settles as Done (`internal/shedadapters/bouncer.go:260`).
A campaign whose live gate is still red then goes on to Describe, PR-Review and Finalize with no parent decision, against D6's "ends capped and its parent decides".
That is folded in here rather than raised alone, because the classification fix and the bound settle the same D9 key set.

### [BLOCKING] The live gate can run no test and still read green

**Location:** `_lym/discussion/decision-record.md:90` (D4), with D2's Preflight at line 54.

**Issue:** lym's hub sets both live keys to `[2]`.
`lym gate test` fills `{test_targets}` with only the packages that hold a test of the tier (`pattern/PATTERN-test-speed.md`, Four tiers).
A campaign over a module whose live tests are tier 1, such as a git-driving one, therefore starts no binary in either live run every round.
D6's CONVERGED then rests on a "green live gate" that verified nothing, and the concurrent amplifier never runs.
Preflight checks that the key exists and does not name tier 3, but not that `packages` holds a test of the key's tiers.

### [BLOCKING] The settled treadle question still stands in Open risks and D8's rationale

**Location:** `_lym/discussion/decision-record.md:200` (D8 rationale), line 286 (Open risks) and `_lym/discussion/support-log.md:89` (Q13).

**Issue:** The operator entry "Delete treadle when Hardener is built (F10)" answers the parent question and replaces the 2026-10-16 decision.
The record still asks it as an open parent question, and the rubric makes such a line a `consistency` finding whose fix drops it.
D8's rationale still carries the misquote the arbiter entry named.
The arbiter's own replacement text has gone stale as well: it calls the deletion "the open parent question" and keeps "re-derived".
The fix therefore quotes the 2026-10-16 note as written and cites the operator entry that replaces it.
It edits neither added entry.
This is raised at BLOCKING because the stale question keeps the Discussion-Review perch escalating a settled parent question every round.

### [MEDIUM] `--copies` still cannot hold its n slots and run its tier inside them

**Location:** `_lym/discussion/decision-record.md:107` (D4).

**Issue:** Carries F4.
`internal/gatecli/run.go:737` acquires each tier run's own slots through `gateslot.Pool.AcquireSlots`.
`run.go:149` skips that acquire only for a slot named in `LYM_GATE_SLOT` of an enclosing process.
A call that already holds every slot of lym's three-slot pool then waits on itself.
The "compiled test binary" wording also departs from the key's tier command and go-json counter (hub `gate.yaml` tier2), which D4's rationale ("the live runs are keys") rests on.

### [LOW] An operator removal of a hardener pair runs no sandbox reap

**Location:** `_lym/discussion/decision-record.md:142` (D5 teardown bound).

**Issue:** Carries F11.
`internal/pairteardown` ends a pair for `lym fabric remove` and Worktree-Teardown, and it runs no reap.
The sandbox hubs' processes are recorded under their own hubs, not under the pair's slug.
The root's nested `.git` keeps `ignoredTaskPathDeletable` refusing the leftover path (`internal/fabricengine/destroy.go`).

## Not raised: below the incremental bar

This round is incremental, so a new finding is raised only at BLOCKING.
These checks held as defects but fall below that bar, and are recorded here for the writer's judgment:
- D8's reinterpretation rewrites only the hub base file (`internal/hubreconcile/reinterpret.go:50`).
  `efforts` is CategoryNone, so a `treadle-judge/<model>` key in a local layer, a run profile or a task's run_settings still fails `checkEfforts` after the role is deleted.
  The D8 criterion holds as written for the hub file only.
- D5 and its criterion commit `selfreport: false`, but loom.yaml has no `selfreport` key: it is retired and ignored.
  Only `friction: ""` turns off the automatic filer.
- D6 does not say when the fix-commits file is written relative to the live gate's re-prompts.
  D7's round-n archive holds the ledger of the judge pass that opened round n, not round n's own verdict.
- D4 step 3's "never runs tier 3" is checked only at Preflight, so a hub edit mid-campaign is not re-checked.
  `--copies` names no bound on its caller or on draining the pool beyond its call.
  The D5 hub list's append names no lock for concurrent `sandbox new` calls.
- D2's check that "the hub's own gate.yaml" sets the live keys meets the template fill and the sparse-base strip for a hub whose live tier is tier 0.
  D2's adjacency refusal and D6's "different model" do not name one comparison for effort-qualified specs.

## Rejected

- interactions' note that HardenFix's verify is a new `verifytree.Verify` site no declared PATTERN change covers.
  HardenFix is the `BurlerFix` constructor whose verify gate already runs through `Verify`, so no PATTERN-gate-slot entry text is broken, and updating the background's site list is Plan's work.

## Focus verdicts

- **D8:** `agentconfig.Load` refuses a retired role in `skills` or in a pair `efforts` key (`config.go` `skillLists`, `checkEfforts`).
  The decided `configreg.Reinterpretation` on the agents module, applied by hubreconcile's walk, follows PATTERN-config-layers.
  The criterion holds for the hub base file, and its reach beyond that file is listed above, below the bar.
- **D4 keys:** both live runs name a gate key and no tier list.
  The serial claim is gone.
  What each key pins can be vacuous (F16).
- **D4 `--copies`:** the top-level call gets its slots; the multi-slot acquire never grants part and clamps above the pool (`pool.go` `takeSlotsInOrder`).
  The tier run inside the call contends with the call's own lease (F18).
- **D5 runproc:** recorded processes end through `runproc.End`, and the `/proc` kill matches the declared change's bound.
  A reed server whose hub is gone is reached through the hub list.
  A hub outside the root is outside the reap by a stated bound.
  Teardown of a root holding repositories is covered for Finalize and the halts, but not for pairteardown (F19).
- **D5, D6, D7 stray trace:** each review-row and fix-row reap appends to the round's one report, and the fix row's end-of-row reap runs before the archive copy, so every row reap reaches round-<n>.
  Finalize's and the halts' reaps run after the last archive, and the record does not say where their output goes.
- **D3 and D9 rotation:** Preflight refuses a one-entry list and adjacent duplicates, a first-last match included.
  Expansion to `hard_max_bounces` keeps CONVERGED reachable past `max_bounces`, and the Bouncer never continues at its hard cap.
- **D4 and D6 red live gate:** a red gate re-prompts the fixer `live_attempts` times (default 2, from hardener.yaml), then ends the row red with the failures deferred, so the campaign ends capped.
  `auto_accept` breaks this (F15).
- **D10:** a segmented build row that no edge from its Bouncer reaches, and one whose segment holds no Bouncer, are each refused with a way forward, and the first criterion names both.
- **D3 and D9 sources:** HardenFix's verify runs `harden-burler-fix`, scoped to the touched trees under `tree_scope`.
  The projection takes the Harden-Review cap from hardener.yaml, `loomCLI.reviewBudget` included.
