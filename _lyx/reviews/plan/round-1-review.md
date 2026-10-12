---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/16-batten-sleep-clock.md:8
    summary: Cards 16, 19 and 21 add a pauserecord import, and card 21 an agentname import, to battenshed, shedverbs and orchengine without editing those packages' import allowlists, so each card fails its own package's AssertImportAllowlistNoStale test.
    fix: "Card 16: add `internal/battenshed#battenshedAllowedImports` under **Edit:**, and in **Intent:** say it gains github.com/Knatte18/loomyard/internal/pauserecord. Card 19: add `internal/shedverbs#shedverbsAllowedImports` under **Edit:**, and in **Intent:** say it gains internal/pauserecord. Card 21: add `internal/orchengine#orchengineAllowedImports` under **Edit:**, and in **Intent:** say it gains internal/pauserecord and internal/agentname. Card 6 already does this for githubclient. All three additions passed `lyx loom validate-plan --overlay` together with no new blocking finding."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/21-orch-pause.md:28
    summary: pauseTick keeps its per-entry state in memory only. A watcher started after a live hub entry's GraceEnd, such as the one lyx orch start spawns for a fresh orch, finds a busy orch the entry does not own and interrupts it, but D4 and D7 say an orch started under a hub entry is not interrupted.
    fix: "Card 21 **Intent:** add a rule under `pauseTick` under a live hub entry: when the watcher first reads an entry whose GraceEnd has already passed and that does not own the orch's session, it marks the entry handled in memory and sends no Esc, so an orch started, or a watcher restarted, after the grace end is never interrupted and gets no resume line. Add a row to TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine: a watcher created after a live hub entry's GraceEnd over a busy orch makes no Interrupt call, still holds notices while the entry is live, and queues no resume line after the resume."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/23-pause-cli.md:54
    summary: The network gate's slug comes from the process location, which is empty in the prime, so batten's network work for a pair (Worktree-Create and Land run in the prime under lyx batten run slug) is not held by a per-run lyx pause slug entry, though D4 says a per-run entry holds network operations run on behalf of its pair.
    fix: "Card 3: add the Create target `plan:internal/pauserecord#SetGateSlug` -> `func SetGateSlug(slug string)`. In **Intent:** it replaces the installed gate's slug, and is a no-op when no gate is installed. Add a row to the Network/ListNetOps test in hold_test.go: a gate installed with an empty slug and then given a slug through SetGateSlug is held by a live entry for that slug. Card 17 **Intent:** `battenCLI.wire` (already a target) calls `pauserecord.SetGateSlug(slug)`, so batten's own remote git and GitHub calls for the slug wait under its per-run entry. This fix passed `lyx loom validate-plan --overlay` with the other target fixes and raised no new blocking finding."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/25-loom-bare-pause-refusal.md:33
    summary: "internal/shedverbs/doc.go line 10 still reads: pause with no flag requests a pause at the next producer boundary. Card 25 makes a flagless pause refuse through the new BarePause hook but leaves that doc out of its targets, though the Constraints list shed's package docs that describe the boundary pause as same-commit edits."
    fix: "Card 25: add `internal/shedverbs/doc.go` under **Edit:**, and in **Intent:** say the doc's pause sentence now reads that a flagless pause sets pause_requested unless the spec's BarePause hook refuses it, which loom's does. This passed `lyx loom validate-plan --overlay` with the other target fixes."
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/22-pause-engine.md:42
    summary: "D4 and the acceptance criteria require lyx pause --status to name each network operation in flight under a parent lock. Card 22 lists network operations apart from readHolds' report, then says only that Status reads the live entries and the same holds report, and its Status test asserts entries and holds only, so --status may omit network operations."
    fix: "Card 22 **Intent:** state that StatusReport carries the same network operations list as PauseReport, from deps.NetOps, marking each one run under a parent lock, beside the live entries and readHolds' report. In resume_test.go's Status case, assert that an open marker under a parent lock is listed and marked."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/26-shed-pause-way-forward.md:43
    summary: The bound card 26 gives for dropping consumeArbiterAttempt's requested-pause refusal, that a loom run can no longer have one requested, is false for a status file written before the upgrade. D8 says such a file still loads, and Relaunch keeps pause_requested.
    fix: "Card 26 **Intent:** replace the clause after the colon with: a pause_requested left by an older binary is no longer refused here; Relaunch carries it through (shedengine keeps pause_requested on a re-launch) and the shed honours it at the next producer boundary, so the consumed attempt still halts there."
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/plan/12-runner-pause-wiring.md:28
    summary: The ImpactSummary says six wiring functions, but the card's targets and Intent tell seven runner sites the geometry, shuttlecli's Command pre-run included.
    fix: "Card 12 **ImpactSummary:** change six wiring functions to seven wiring sites (six wire functions and shuttle's Command pre-run)."
---

### [BLOCKING] Import allowlists not edited where cards add imports

**Location:** `_lyx/plan/16-batten-sleep-clock.md:8`, `_lyx/plan/19-watchdog-awake.md:6`, `_lyx/plan/21-orch-pause.md:10`

**Issue:** Three packages pin their production imports with `scankit.AssertImportAllowlistNoStale`: `internal/battenshed/seam_enforcement_test.go`, whose `battenshedAllowedImports` holds only shedengine, logger and parentlock; `internal/shedverbs/seam_enforcement_test.go` (`shedverbsAllowedImports`); and `internal/orchengine/seam_enforcement_test.go` (`orchengineAllowedImports`).
None of these lists admits `internal/pauserecord`, and orchengine's also lacks `internal/agentname`.
Card 16 makes battenshed import `pauserecord.SleepWatch`, `AwakeSince` and `Interval`.
Cards 19 and 20 make shedverbs import `pauserecord.SleepWatch`, `AwakeSince`, `MonotonicAwake` and `Coverage`.
Card 21 makes orchengine import `pauserecord` and call `agentname.Parse`.
Each card leaves its package's allowlist out of its targets, so its package test fails on that card's own gate.
Card 6 handles the same case for `internal/githubclient#allowedImports`, which shows the method: an allowlisted package that gains an import edits its list on the same card.
The gap recurs across three cards, so it is filed once.

### [MEDIUM] A watcher started after the grace end interrupts a freshly started orch

**Location:** `_lyx/plan/21-orch-pause.md:28`

**Issue:** `pauseTick` keeps "once per entry" state in memory, keyed by `PausedAt`.
Its only restart guard is an entry that already owns the orch's session.
`lyx orch start` spawns a watcher for a fresh session (`internal/orchcli/start.go`: "spawns a watcher for it").
That new watcher reads a live hub entry that is past `GraceEnd` and does not own the orch, finds the orch busy with the operator's first turn, and plays the interrupt choreography up to twice.
D4's table says `lyx orch start` is not held and that "an orch started under a hub entry is not interrupted", and D7 repeats the case.
No card prevents this, and card 21's test has no row for it.

### [MEDIUM] Batten's prime-side network work escapes a per-run pause

**Location:** `_lyx/plan/23-pause-cli.md:54`

**Issue:** `InstallNetworkGate` builds the gate from `hubgeom.PauseGeometry` of the process's location.
Card 4 sets the slug to empty for the prime.
`lyx batten run <slug>` runs in the prime, and so do its own Go rows that reach the network: `Worktree-Create` and `Land` in `contracts/recipes/batten-recipe.yaml`, with Land holding the hub-level land lock rather than a parent lock (`internal/battenshed/land.go`).
Under `lyx pause <slug>`, `WaitUncovered` is called with an empty slug, and `Record.Covering("")` is covered by a hub entry only (card 1).
So batten's pushes and fetches on behalf of the covered pair go through, though D4 says "a per-run entry holds those run on behalf of that pair, except one whose caller holds a parent lock".
Only the hub-wide pause holds them.

### [MEDIUM] shedverbs package doc still describes the bare pause

**Location:** `_lyx/plan/25-loom-bare-pause-refusal.md:33`

**Issue:** `internal/shedverbs/doc.go:10` reads "pause with no flag requests a pause at the next producer boundary."
After card 25, a flagless pause refuses whenever the spec sets `Hooks.BarePause`, which loom always does.
The Constraints section's same-commit list includes "the batten, shed, … package docs that describe the boundary pause".
Card 25 edits the shedverbs pause body and hook but not the doc that describes them.
Cards 19 and 20 edit that doc only for the watchdog.

### [MEDIUM] `lyx pause --status` may not name network operations

**Location:** `_lyx/plan/22-pause-engine.md:42`

**Issue:** Step 4 of card 22's Pause report lists "network operations in flight, naming those under a parent lock" separately from "(`readHolds` over `Locks`)".
`Status` is then defined as "the live entries and the same holds report".
The text admits a StatusReport with no network operations, and no test pins them in Status: card 22's Status case asserts entries and holds only, and card 23's `--status` case asserts the live entry only.
D4 says "`lyx pause --status` name[s] each as a network operation in flight under a parent lock", and the acceptance criterion "the pause report and `lyx pause --status` name it" repeats it.

### [LOW] The arbiter-clause removal states a false bound

**Location:** `_lyx/plan/26-shed-pause-way-forward.md:43`

**Issue:** D8 licenses dropping the requested-pause clause, so the change itself is in scope.
But the Intent bounds the removed guard with "a loom run can no longer have one requested".
D8's acceptance says "a status file carrying `pause_requested` still loads", and `internal/shedengine/doc.go:68` says Relaunch keeps `pause_requested` across a re-launch.
A run upgraded mid-flight can therefore carry one.
The real bound is that Relaunch keeps the flag and the shed honours it at the next boundary, and that is what the Intent should state.

### [NIT] Card 12's ImpactSummary undercounts its sites

**Location:** `_lyx/plan/12-runner-pause-wiring.md:28`

**Issue:** The card lists seven runner sites: `loomCLI.wire`, `battenCLI.wire`, both of burler's wire functions, both of webster's, and `shuttlecli#Command`'s pre-run.
The summary says "six wiring functions".

## Rejected

- Card order (focus 1): no card uses a symbol a later card creates.
  Cards 5 and 6 call only `pauserecord.Network` and `InstallGate` from card 3, never `InstallNetworkGate` from card 23, and their tests install a gate directly.
  Card 12 uses `Runner.SetPause` and `PauseGeometry` from card 8.
  Cards 8 to 20 use geometry from card 4, sleep and record symbols from cards 1 to 3, and the step tie from card 10 before card 20.
  Card 11's mention of card 20 is prose only.
- Shared Decision `pause-hub-resolution` (focus 2): the departure from D1's "through `internal/cliwire`" is grounded, not a defect.
  `PATTERN-cliwire-sole-wiring` scopes cliwire to the standalone-capable CLIs, `internal/cliwire`'s seam test denies `internal/lyxcwd`, and `cleanupcli` resolves a hub-only CLI with `lyxcwd.Resolve` the same way.
- Placing the git hold in `gitexec.runCore` (card 5) rather than in fabricengine's own files is still "inside" the stack beneath fabricengine and never at its callers, and the Shared Decision records why. It does not breach `PATTERN-fabric-git` or the Constraint.
- Reachability (focus 4): cards 13, 15, 25 and 26 each name an input that passes every upstream check (argument checks; no earlier check in start, resume or batten preRun; the absent-file check in `shedverbs` pauseCmd) and a test that reaches the refusal.
- `## verify:` (focus 7) runs `./...` untagged and under `-tags integration`, and vets the tmux and llm tags, in the stencil's order.
  The only nested module, `internal/planglyph/testdata/glyphchain`, is test data and no card targets it.
- Reworded messages: every test asserting the old busy way forward (`TestStep_ErrShedBusy`, `TestGoto_RunLockHeldRefusesAndLeavesFile`, `TestGoto_RunningRunRefusedUntilPaused`, `TestRelaunch`, shedverbs' goto tests and `TestTeardown_CallSequence`) is on card 26.
- Test economy: the new test functions each pin a behavior no existing test covers (checked against the parentlock, githubclient, shuttleengine, battencli and loomcli test sets).
  Card 7's covered yield is a different grant condition from `TestTryAcquire_GrantsWaitersInTicketOrderSkippingDeadOnes`'s liveness skip.
- Card 23's sandbox tag `**Covers:** pause, resume` is right: `cmd/lyx/sandbox_coverage_test.go` treats every root child as a module, `resume` included.
