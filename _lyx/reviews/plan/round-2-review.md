---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/21-orch-pause.md
    summary: The orch watcher skips any hub entry whose GraceEnd has passed when it first reads it, so a live watcher never interrupts a busy orch under --grace 0 or when poll_interval_ms is at least the grace, against D7
    fix: "In card 21's pauseTick bullets, replace the rule 'an entry whose GraceEnd has already passed when the watcher first reads it ... is marked handled in memory with no Esc' with one keyed on the watcher's own start: Watcher records the time of its first tick (a new in-memory field on Watcher, already an Edit target), and only an entry whose GraceEnd lies before that first tick is marked handled with no Esc; an entry the watcher was already running for, including one whose GraceEnd equals PausedAt, is handled by the before-GraceEnd and at-GraceEnd bullets even when the first tick that reads it falls after GraceEnd. Add rows to TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine: a watcher created before the entry, with the entry written at grace 0 and the first tick reading it after GraceEnd, interrupts a busy orch and records it; the existing row for a watcher created after a live entry's grace end stays."
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/plan/21-orch-pause.md
    summary: An orch started under a live hub entry before its grace end, and busy with its first turn, is interrupted at GraceEnd, against D4's row that an orch started under a hub entry is not interrupted; no test row reaches the case
    fix: "Add `internal/orchengine#State` and `internal/orchengine#ResetForFreshLaunch` to card 21's Edit list. State gains LaunchedAt (JSON launched_at), set by ResetForFreshLaunch from its launchedAt argument, which every fresh launch and adoption already passes. In pauseTick, an entry whose PausedAt is not after st.LaunchedAt (a zero LaunchedAt from an older state file reads as launched before every entry) is the orch-started-under-the-entry case: nothing is interrupted, nothing is recorded, and no resume line goes. Add a row to TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine: a state whose LaunchedAt lies between the entry's PausedAt and GraceEnd, with a busy probe, gets no Interrupt call at the grace end and no resume line after the resume. Name the rule in the card's Bound line."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/23-pause-cli.md
    summary: Card 23 adds sandbox scenario S11, but tools/sandbox/SANDBOX-CORE-SUITE.md already has S11 (Routine cleanup as a dry run), so the suite would hold two S11 scenarios
    fix: In card 23's Intent, name the new scenario S12 instead of S11, and add that it also gains an S12 line in the suite's Session log format block after the S11 line.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/17-batten-pause-reader.md
    summary: No test reaches battenCLI.wire's pauserecord.SetGateSlug call, the one place that makes a per-run entry hold batten's Worktree-Create and Land network calls in the prime
    fix: "Add to card 17's Create list `plan:internal/battencli#TestWire_PointsTheNetworkGateAtItsSlug` -> `func TestWire_PointsTheNetworkGateAtItsSlug(t *testing.T)` and add `cmd/lyx/testdata/test-budget.yaml` to its Edit list. The test (not parallel, since the gate is process-wide) installs a pauserecord.Gate with an empty slug over a temp record holding a live entry for the slug, with a Sleep that records its call and returns an error, wires a battenCLI for that slug as wiredPrime does, calls pauserecord.Network, and asserts Sleep was called; it restores the gate on cleanup. The card's Tests list names it and the battencli budget row grows by one."
---

### [BLOCKING] A live orch watcher misses a hub entry whose grace end has passed by its first read

**Location:** _lyx/plan/21-orch-pause.md

**Issue:** Card 21's third `pauseTick` bullet marks an entry handled with no Esc when "`GraceEnd` has already passed when the watcher first reads it".
That condition is meant to catch a watcher started or restarted after the grace end, but "first reads it" is per entry, not per watcher process.
A watcher that has run for hours reads a new entry first on its next tick after the entry is written.
D1 admits `--grace 0`, so `GraceEnd` equals `PausedAt`, and every live watcher's first read of that entry falls after `GraceEnd`.
The busy orch is then never interrupted, even though the pause report names its case as delegated to a live orch watcher.
The same happens whenever `poll_interval_ms` in `internal/orchengine/template.yaml` (default 2000, configurable upward) is at least the grace.
This contradicts D7 ("At that grace end it plays the interrupt choreography into a busy orch") and the acceptance criterion that a busy orch under a hub entry is interrupted at the entry's recorded grace end.
The card's test rows cover only a watcher created after the grace end, so no row catches the live-watcher case.

### [MEDIUM] An orch started inside a hub entry's grace window is interrupted

**Location:** _lyx/plan/21-orch-pause.md

**Issue:** D4's table says `lyx orch start` is not held: "an orch started under a hub entry is not interrupted".
Card 21 protects only an orch whose watcher first reads the entry after `GraceEnd`.
Suppose the operator runs `lyx orch start` within the grace (10 s by default, up to 60 s).
Then the watcher, whether `start` spawned it fresh or it was already live, reads the entry before `GraceEnd`.
The orch is busy with its start turn, so the at-`GraceEnd` bullet records it and sends Esc into it.
Nothing in `pauseTick` can tell this orch from one that was running when the entry was written.
`orchengine.State` carries no launch time the watcher could compare against `PausedAt`; `ResetForFreshLaunch` already receives `launchedAt` but stores it only as `CompactionBaseline`, which compaction moves.
No row of the card's new test starts the orch between `PausedAt` and `GraceEnd`.
This answers the focus directive's second question: a watcher started after the grace end does not interrupt, and that case has a test row.
An orch started before the grace end is interrupted, and that case has no row.

### [LOW] Card 23's sandbox scenario reuses S11

**Location:** _lyx/plan/23-pause-cli.md

**Issue:** Card 23 says `tools/sandbox/SANDBOX-CORE-SUITE.md` "gains scenario S11 tagged `**Covers:** pause, resume`".
That file already has `### S11 -- Routine cleanup as a dry run` (`**Covers:** cleanup`), and its Session log format block ends at S11.
Two scenarios would then share a number, and the session log would have one line for both.

### [LOW] No test pins the batten gate slug that holds Worktree-Create and Land

**Location:** _lyx/plan/17-batten-pause-reader.md

**Issue:** This settles the third focus question.
`pausecli.InstallNetworkGate` (card 23) takes the gate's slug from the process location, which is empty in the prime.
Card 17's `battenCLI.wire` calls `pauserecord.SetGateSlug(slug)`.
`wire` runs in `armAt` (`internal/battencli/arm.go`) before the shed walks any row.
The root's pre-run runs before batten's, because `cmd/lyx/main.go` sets `cobra.EnableTraverseRunHooks`, so the gate is already installed when `wire` sets the slug.
Worktree-Create, Land and Worktree-Teardown run in-process in that `lyx batten` process, and their fabricengine remote calls go through `gitexec.runCore` (card 5), so a per-run entry for the slug does hold them.
Card 17 also skips the arm step while covered, so `ParentMoved`'s fetch through `ReadParentTips(…, true)` is not reached under a pause.
The design is sound, but no test reaches the `SetGateSlug` call.
Card 3's test pins `SetGateSlug` itself, and card 23's test pins `InstallNetworkGate` for a pair location.
If the call in `wire` were dropped, every test would still pass, and batten's prime-side network calls would silently stop being held by a per-run entry.
D4's acceptance criterion that a fabric push or fetch waits under a covered run would then be unmet with no test failing.

## Rejected

- Import allowlists (first focus question): no card makes a package fail its own allowlist test.
  The packages that pin imports with `scankit.AssertImportAllowlistNoStale` or `AssertImportAllowlist` and gain an import are `internal/githubclient` (card 6 lists `allowedImports`), `internal/battenshed` (card 16 lists `battenshedAllowedImports`), `internal/shedverbs` (card 19 lists `shedverbsAllowedImports`) and `internal/orchengine` (card 21 lists `orchengineAllowedImports`, adding `pauserecord` and `agentname`, neither listed today).
  `internal/parentlock` (card 7 adds a `func(string) bool` field), `internal/landingshed` (card 27), `internal/shedengine` and `internal/shedbuild` (card 26) gain no import.
  The scan is AST-direct, so `cliwire`'s allowlist is unaffected by `standalonegeom` itself importing `pauserecord`.
  The packages that gain `pauserecord` without an allowlist (`gitexec`, `hubgeom`, `standalonegeom`, `reedengine`, `shuttleengine`, the `*cli` packages) carry no import-allowlist test; `gitoracle`'s list pins only its own direct imports.
- Orch restart after the grace end: card 21's row "a watcher created after a live entry's grace end over a busy orch makes no `Interrupt` call ... and queues no resume line" reaches that case; the defects are F1 and F2.
- `lyx resume` as a second root command rather than a `pause` subcommand registered twice: `cmd/lyx/sandbox_coverage_test.go` keys coverage on `newRoot().Commands()`, and card 23's `**Covers:** pause, resume` covers both, so this is not raised.
