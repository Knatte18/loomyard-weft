---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md:182
    summary: The constraint that the gate never verifies under a running fork is false; after the first gated Done both output files stay on disk, so Merriam's turn end after spawning the fixer re-arrives at the gate while the fork edits the tree
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:86
    summary: D5's non-merge rule fails the gate Terminal on a clean parent merge-in landing during the fix window, which record-batch already accepts as legitimate mid-run work
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:86
    summary: D5's no-plan-or-_lyx-path commit check cannot fire, since the plan and _lyx live in the weft worktree outside every warp commit, so the stated bound on the fixer's plan writes is illusory
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:111
    summary: D7's skip bound assumes only the shared function writes the verified-tree record, but nothing states who may write it, and the clean-tree check never sees a write under the ignored .lyx
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:115
    summary: D8 adds a Stuck edge that carries a red tree to Webster-Bouncer but states no bound; what stops the red tree lives only in Open risks
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md:34
    summary: D1 does not state that the per-card gate is prompt-rendered and never run by Go, nor what bounds a fork that skips it; that bound lives only in Open risks
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md:37
    summary: D1 runs integration-tagged tests per card, which contradicts loom-plan-spec Tier 2's once-per-plan rationale, yet the spec update is scoped as Tier 1 becoming implemented
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md:54
    summary: D3 step 1 passes on an absent outcome.yaml without stating what bounds that pass-through
  - id: F9
    severity: LOW
    location: _lyx/discussion/decision-record.md:203
    summary: Acceptance criteria miss D2's Plan-Write grep rule, D6's blocking recover-batch and poll_wait_s degradation, and D4's verify-gate.yaml report
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md:75
    summary: D4's rationale carries incident narrative (card 07 named for card 09's line) that the support log's Interview item 3 already records
---

### [BLOCKING] The gate re-arrives while the fixer fork works

**Location:** `_lyx/discussion/decision-record.md:182` (Constraints), with D3 (line 49) and D5 (line 78).

**Issue:** The Constraints entry says "The gate closure runs only at a turn boundary with no background work outstanding, so it never verifies under a running fork."
The code does not behave that way.
`pollEventsTick` (`internal/shuttleengine/wait.go`) returns `OutcomeDone` whenever every `OutputFiles` entry exists, and checks `EventWaiting` only after that.
Merriam's spec names `outcome.yaml` and `summary.md` as its output files (`internal/websterengine/runlevel.go`).
Merriam writes both before its first Done arrival, so both still exist after the gate's first failure.
D5 then has Merriam spawn a background fixer fork and end its turn.
That Stop line carries outstanding background work, and claudeengine parses it as `EventWaiting`, but `pollEventsTick` still classifies it Done because the files exist.
The gated Done branch then runs the gate with the fixer mid-edit.
The clean-tree check fails on the fork's uncommitted edits, which spends an attempt and re-prompts Merriam while its fork is still running.
Three such turn ends exhaust the gate before any fix lands.
The plan writer reads this as a fact about the existing loop, not as a requirement, so nothing in the plan would make it true.

**Fix:** Make it a requirement in D3: on a gated run, a turn end with background work outstanding (`EventWaiting`) is not a gate arrival, even when every output file exists.
State the bound: the deferral applies only while the session is live, and the deadline and liveness exits still classify Done from the files (Completion Signal Invariant), so `master_timeout_min` still bounds a fork that never finishes.
Remove the Constraints entry, which becomes D3's own text, and add an acceptance test: a turn end that spawns the fixer in the background does not evaluate the gate.

### [MEDIUM] D5's non-merge rule refuses a legitimate parent merge

**Location:** `_lyx/discussion/decision-record.md:86`

**Issue:** D5 requires every commit from the pre-fix head to HEAD to be a non-merge commit, and fails the gate `Terminal` otherwise.
Parent merge-ins land mid-run: `internal/websterengine/doc.go` documents that HEAD may sit above a batch's `head_sha` by clean parent merges alone on the first-parent chain, and `parentmerge_test.go` pins the 2026-09-30 regression where a merge-in landed between a fork's commit and `record-batch`.
A clean parent merge that lands during the fix window is legitimate work, and D5 would halt the run for a human on it.
The other writes after a gate failure do not conflict with the check.
Merriam's `outcome.yaml` and `summary.md` rewrites land in `_lyx/webster`, which is in the weft worktree and never in a warp commit.

**Fix:** Accept clean parent merges on the first-parent chain, as record-batch already does, and refuse any other merge commit.
Add the case to the commit-check acceptance test.

### [MEDIUM] D5's path check names paths no warp commit can contain

**Location:** `_lyx/discussion/decision-record.md:86`, with the acceptance criterion at line 210.

**Issue:** `_lyx` is a symlink into the separate `-weft` worktree, excluded from the warp repository (`.git/info/exclude`), so no warp commit can touch a plan or `_lyx` path.
In standalone geometry `WorktreeRoot` is the target and is disjoint from the state tree (`internal/websterengine/geometry.go`).
The check "touching no plan or `_lyx` path" therefore never fires in either geometry.
D5's bound says fixer commits are "bounded by that commit check", which leaves the fixer's writes to the plan directory unbounded in fact.
The criterion "a fixer commit touching the plan directory fails the gate `Terminal`" describes a case production geometry cannot produce.
The guard that sees such writes is the fork transcript audit (`CheckFork`, which takes the plan and webster dirs), which `runExitAuditCrossCheck` applies to out-of-batch forks through `forkOwnReport`.

**Fix:** Rest D5's bound on the run-exit fork audit, which must refuse a fixer fork's write to the plan or webster dirs.
Narrow the commit check to the shape F2 settles.
Change the acceptance criterion to a fixer fork that writes the plan directory and is refused by the run-exit audit.

### [MEDIUM] D7 does not bound who writes the verified-tree record

**Location:** `_lyx/discussion/decision-record.md:111`, with D15 (line 165).

**Issue:** The record lets every verify site skip.
D7's only bound is the tree match: "the skip lets through only a tree byte-identical to one that passed the same command".
That holds only if nothing but the shared function writes the record.
The record lives under `.lyx`, which is gitignored, so D7's own clean-tree check (ignored files excluded) never sees a hand-written one.
Merriam, the fixer fork, a Burler round session and the conflict session all run with file-write tools in that worktree.
D15's verb writes the record too, though only through the function after a pass, so it keeps the bound.

**Fix:** State in D7's bound that only the shared function writes the record, so `lyx webster verify` records only after a pass of its own.
State also that no agent prompt names the record path, and that an agent writing it is outside the bound: a steering guard of the same class as the other agent write bans, not a security boundary.

### [MEDIUM] D8's Stuck edge states no bound

**Location:** `_lyx/discussion/decision-record.md:115`

**Issue:** An exhausted Webster-Burler `verify` gate routes Stuck to Webster-Bouncer, carrying a tree that failed verify.
D8 states no bound on that edge.
What stops the red tree is in Open risks only ("Publish's verify then halts it").
The bound holds through D7: a failing verify writes no record, so Publish's verify runs on that tree.
The plan writer gets that as a risk note, not as a requirement to keep.

**Fix:** Add a Bound line to D8: an exhausted gate writes no record, so a red tree Webster-Bouncer approves reaches Publish, whose D7 verify runs because the tree differs from the record, and Stuck halts it before any push.
Shorten the Open risks entry to the remaining risk and drop the restated bound.

### [LOW] D1 does not state that the card gate is unenforced

**Location:** `_lyx/discussion/decision-record.md:34`

**Issue:** D1 says `begin-batch` renders the command into the fork prompt, but not that Go never runs it.
D1's rationale cites the spec's "mechanical per-card run", which a plan writer could read as Go enforcing it.
The bound on a fork that skips the gate (the webster gate, localized by D4's hint) is in Open risks (line 198) only.
This is not a new skip, because today's card gate is also prompt text, but the Decision should say what it is.

**Fix:** Add a Bound line to D1: the fork runs the gate and Go does not, so a fork that skips it is caught at the webster gate, which localizes through D4's hint.
Drop the Open risks entry that restates it.

### [LOW] D1 contradicts the spec's Tier 2 rationale without saying so

**Location:** `_lyx/discussion/decision-record.md:37`, with Notes for the plan writer (line 236).

**Issue:** `contracts/specs/loom-plan-spec.md` scopes Tier 1 to untagged per-package `go test` and says of Tier 2: "paying that cost once per plan rather than once per card that happens to touch an affected package is the point."
D1 runs `go test -tags integration` over the card's own packages on every card, so it moves part of Tier 2 to per card.
The Notes say only "Tier 1 becomes implemented as D1", so the spec update would leave Tier 2's rationale contradicting the code.

**Fix:** State in D1 that the card gate also runs the integration-tagged tests of the card's own packages, which the spec's Tier 2 text must now describe, and that the plan-level `## verify:` stays the once-per-plan run.

### [LOW] D3 step 1's absent-outcome pass is unbounded on its face

**Location:** `_lyx/discussion/decision-record.md:54`

**Issue:** Step 1 passes without verifying when `outcome.yaml` is absent and states a consequence only for a stuck or paused outcome.
The bound exists in code but is unstated.
A Done arrival requires every output file (`pollEventsTick`), so an absent `outcome.yaml` cannot reach the gate from a live arrival.
If it ever did, `mapMasterDone`'s `parseOutcome` refuses the missing file after the gate, and the run returns an error with no Publish.

**Fix:** State in step 1 that an absent `outcome.yaml` cannot reach the gate, and that if it did, the run errors at the post-gate outcome parse.
State also that a non-`done` outcome ends the run without reaching Publish.

### [LOW] Acceptance criteria miss observable Decisions

**Location:** `_lyx/discussion/decision-record.md:203`

**Issue:** D2's grep rule is a stencil change with no criterion.
The stencil-absence criterion covers D6's removed loop but not the new behavior: `recover-batch` blocking to a terminal digest, and a `webster.yaml` still carrying `poll_wait_s` loading.
D4's per-run `verify-gate.yaml` report, which replaces `integration.yaml`, has no criterion.

**Fix:** Add criteria: a template test pins D2's grep rule in `loom-template-plan.md`; `recover-batch` returns only at a terminal digest or at `recovery_timeout_min`; a `webster.yaml` with `poll_wait_s` loads; a failed gate writes `verify-gate.yaml` with the identities, hint and fix commits.

### [NIT] Incident narrative in D4's rationale

**Location:** `_lyx/discussion/decision-record.md:75`

**Issue:** "That is how it named card 07 for card 09's line." is incident narrative.
Under the completeness-before-leanness test it carries no requirement: the structural causes stated before it carry the reason, and the Decision already states that bisect goes.
The support log's Interview item 3 records the incident.

**Fix:** Delete the sentence from `decision-record.md`.

## Focus coverage

- D3 steps 1 and 4: step 1 is F8 and F1.
  Step 4's no-`## verify:` pass is today's behavior: landing's verify gate also skips with a warning when no command is configured, and the step states its consequence. Not a defect.
- D7 and D15: F4.
- D8: F5.
- D1: F6, and F7 found alongside it.
- D5: F2 and F3. The `outcome.yaml` and `summary.md` rewrites do not conflict with the check, since they are weft-side.
- D3 step 5 and attempt counts: not a defect.
  The flaky pass is stated in D3 with its bound (one rerun of the same command, warning, summary section, friction note), and a rerun pass records through D7 like any pass.
  The attempt reset after an attach only adds attempts to a must-pass entry and never lets a red tree through.
  It sits in the decision record's Constraints, which Plan-Write reads.
- Acceptance criteria: F9.
- Relocation: D4's narrative is F10.
  D7's board-entry reference is one clause of provenance beside the reason it gives, not narrative, so it stays.
  The Notes for the plan writer name modules and symbols, never signatures, `file:line` citations or interface shapes, so they stay inside the stencil's Step 2 bound.
