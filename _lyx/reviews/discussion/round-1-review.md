---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (Open risks, first bullet; Scope Out, first bullet)
    summary: The reed-add-parentless-placement risk rests on a false premise, since that task sweeps only the two reed suites and never the shuttle suite, and it misses the real risk that its D4 retires the stacked shell-window layout C13 depends on
    fix: In decision-record.md Open risks, replace the reed-add-parentless-placement bullet with one stating that task's facts as read from its plan, namely that its sandbox sweep (card 8) names only SANDBOX-REED-SUITE.md and SANDBOX-REED-WATCH-SUITE.md, so it edits none of the moved lines whichever task lands first, and that its D4 removes the shell window's stacked strand layout, so after it lands C13's stacked 3-row strand may no longer be reachable and C13 then needs a follow-up outside both tasks. In Scope Out, reword the clause that says the issue-673 fix owns them to say that the plain parentless lyx reed add lines need no --anchor once issue 673's default changes, and drop the ownership claim.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D1, Controlled tmux exceptions bullet; Scope In, first bullet)
    summary: D1 carries a non-black-box tmux list-panes exception into the creel suite although no creel scenario uses it, and its only stated bound reads the socket from a shuttle run's strand guid, which no creel scenario has
    fix: In D1, delete the Controlled tmux exceptions bullet and say instead that the creel suite has no controlled tmux exceptions section, because C1's read-only socket-directory listing is already in the controlled creel exceptions and no S9 or C scenario lists panes; in Scope In's first bullet change controlled tmux and creel exceptions to controlled creel exceptions; add to support-log.md's Rejected alternatives a line that carrying the shuttle tmux exception was rejected because no creel scenario needs it and its socket lookup depends on a shuttle run.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1, Fingerprint header / how to run / verdict key / capturing findings bullet)
    summary: D1's substitution of lyx creel for lyx shuttle leaves the how-to-run discovery line naming only lyx creel while D1's own black-box rule names lyx creel and lyx reed, and leaves the capturing-findings JSON example with ref S2, an id the creel suite does not hold
    fix: Reword that D1 bullet to say the how-to-run discovery line names lyx creel and lyx reed with their --help, as the black-box rule does, and that the capturing-findings JSON example's ref names a creel id such as C2.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md (D1, Pre-conditions bullet)
    summary: The shuttle suite's live-runtime pre-condition requires tmux, PowerShell 7 and a logged-in claude, but D1 lists only live tmux and claude, leaving open whether PowerShell 7 carries into the creel suite
    fix: In D1's Pre-conditions bullet, state whether PowerShell 7 is a creel pre-condition; if it is, add it beside live tmux and the logged-in claude.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (Open risks, creel daemon bullet)
    summary: The creel daemon risk says its open lock and socket may block build.cmd -reset, but internal/creelengine/doc.go puts the socket in the user's lyxtemp root outside the hub; only the lock and daemon log sit in the hub's creel state directory
    fix: In that Open risks bullet, replace its open lock and socket with its open lock and log file in the hub's creel state directory.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, shuttleSuite keeps reedTeardown bullet); _lyx/discussion/support-log.md (Turn 2, scenarios LOW nothing pins shuttleSuite)
    summary: The shuttleSuite teardown criterion rests on the lens claim that nothing pins it, but TestSuiteSpecs_ReedTeardownFlag in tools/sandbox/suite_test.go already asserts shuttleSuite.reedTeardown, and no Decision touches shuttleSuite's spec
    fix: Delete the criterion shuttleSuite keeps reedTeardown true from decision-record.md, and in support-log.md change that Turn 2 entry from fixed, criterion added to rejected, TestSuiteSpecs_ReedTeardownFlag already pins it and no Decision changes shuttleSuite.
---

### [MEDIUM] The parentless-placement open risk rests on a false premise and misses the C13 risk

**Location:** `_lyx/discussion/decision-record.md`, Open risks first bullet and Scope Out first bullet.

**Issue:** The risk says `reed-add-parentless-placement` "sweeps parentless `lyx reed add` lines, which S9, the creel fixtures and C13 hold", and that its edits either carry into the new file or "that sweep covers the new file".
That task's own decision record (D8) and plan card `08-sweep-docs-and-sandbox.md` name only `tools/sandbox/SANDBOX-REED-SUITE.md` and `tools/sandbox/SANDBOX-REED-WATCH-SUITE.md`; neither touches `SANDBOX-SHUTTLE-SUITE.md`, and S9's and C13's lines hold no `below-parent` spelling for its grep sweep to hit.
So no edit carries over in either landing order, and nothing covers the new file.
The real cross-task risk is the opposite one: that task's D4 removes the shell window's stacked strand layout, and C13's goal is to post "to one squeezed into a stacked 3-row pane".
After it lands, C13's stacked half may have no reachable setup, and neither task updates it.
Scope Out's "the #673 fix owns them" leans on the same false premise; the conclusion (no `--anchor own-window`) holds only because the fixed default makes the plain lines correct.

### [MEDIUM] D1 keeps an unused tmux exception whose bound only applies to shuttle

**Location:** `_lyx/discussion/decision-record.md`, D1 "Controlled tmux exceptions" bullet, and Scope In first bullet.

**Issue:** The shuttle suite's tmux exception allows `tmux -L <socket> list-panes`/`ls` "where `<socket>` is read from the shuttle run's strand guid cross-referenced against `lyx reed status` output".
No S9 or C scenario lists panes; the only tmux-adjacent step, C1's read-only socket-directory listing, is already in the controlled creel exceptions, which D1 moves verbatim.
Carrying the tmux exception widens the creel suite's non-black-box surface with nothing that needs it, and its one stated bound, the shuttle run's strand guid, does not exist for hand-added creel strands, so the writer must invent a bound.
The discussion spec's compaction rules require an escape hatch's Decision to state its bound; this one has none that applies.
The board entry's Idea names "controlled tmux and creel exceptions", but as the agent's suggestion, not the operator decision, so dropping it is open to the run.

### [LOW] D1's substitution rule leaves shuttle-only text in the creel preamble

**Location:** `_lyx/discussion/decision-record.md`, D1 bullet on fingerprint header, how to run, verdict key and capturing findings.

**Issue:** The shuttle suite's how-to-run section says to discover commands "via `lyx shuttle`, `lyx shuttle <subcommand>`, and `--help`"; substituting `lyx creel` alone contradicts D1's black-box bullet, which names `lyx creel` and `lyx reed`, and S9 and the fixtures need `lyx reed add`.
The capturing-findings JSON example carries `"ref": "S2"`, which the substitution leaves in place though the creel suite holds no S2.

### [LOW] D1 is silent on PowerShell 7

**Location:** `_lyx/discussion/decision-record.md`, D1 Pre-conditions bullet.

**Issue:** `SANDBOX-SHUTTLE-SUITE.md` pre-condition 3 lists "tmux (or the Windows tmux port) on PATH, PowerShell 7, and a logged-in `claude` on PATH".
D1 names live tmux and a logged-in `claude` and drops PowerShell 7 without saying whether creel needs it, so the plan writer either guesses or copies it.

### [LOW] The daemon open risk misplaces the socket

**Location:** `_lyx/discussion/decision-record.md`, Open risks, creel daemon bullet.

**Issue:** `internal/creelengine/doc.go` says "The socket sits in the user's lyxtemp root", outside the hub, so it cannot block removing the hub.
`internal/creelengine/geometry.go` puts `LockPath` and `DaemonLogPath` in `StateDir`, `<hub>/_board/.lyx/creel`, which is what a Windows `build.cmd -reset` can trip on.
The rest of the bullet holds: the doc says a replaced binary ends the daemon.

### [LOW] The shuttleSuite teardown criterion answers a false lens premise

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria; `_lyx/discussion/support-log.md`, Turn 2.

**Issue:** The support log records "scenarios LOW, nothing pins `shuttleSuite` keeping teardown: fixed, criterion added".
`TestSuiteSpecs_ReedTeardownFlag` already loops over `reedSuite`, `shuttleSuite` and `burlerSuite` and fails on any with `reedTeardown` false.
No Decision touches `shuttleSuite`, so the criterion adds no behavior to cover and only grows the record.

## Verified, no finding

- D1/D3 split against the shuttle suite text: the S2 operator-attach line, the controlled creel exceptions (C1, C10, C12), the `ref` list `S1`-`S9`, `C1`-`C13`, the session log listing S1, S2, S3, S8, S9 and C1 to C13 (so D3's S4 to S7 claim is true), and the Notes creel line all match what D1 and D3 say; no requirement is left in both files beyond F2.
- D3's shuttle Notes pointer does not contradict Scope Out, which excludes only the sibling suites' Notes lines.
- D4's bound holds against `runSuite` in `tools/sandbox/suite.go`: it writes only `spec.fileName`, removes a stale `sandbox-report.json`, git-excludes both names, launches the agent, runs `reedDown` after it when `reedTeardown` is set, and returns the `run sandbox/build.cmd first` error before any write when the warp repo is absent.
  Every D4 behavior has a criterion, and each maps onto an existing table (`TestRun_SuiteSubcommands`, `TestRunSuite_PerSuiteSpec`, `TestSuiteSpecs_ReedTeardownFlag`) as a row.
- D5's premise is true: `parseCoveredModules` in `cmd/lyx/sandbox_coverage_test.go` globs `tools/sandbox/*SUITE.md` and needs at least two files; `tools/sandbox` has a row in `cmd/lyx/testdata/test-budget.yaml`, and rows in existing tables add no top-level test.
- No criterion or Decision edits this task's own board entry.
