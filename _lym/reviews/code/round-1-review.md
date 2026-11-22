---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: assist:assist
    location: internal/reedengine/lifecycle.go:882
    summary: An orch strand launched under the old default replays its stored resume line, with remote control on, when reed revives its pane, so lym still turns remote control on without the hub's config asking.
    fix: "Leave the code in place and defer this finding to the run's parent. The board entry asks that no hub gets remote control on any launch or resume unless its config turns it on, and the summary's 'Other launch paths' section accepts a transition window instead: reed's Resume (internal/reedengine/lifecycle.go:882) and its absent-pane relaunch (internal/reedengine/strand.go:615) replay the ResumeCmd stored at the strand's launch. The parent either records acceptance of that window with `lym loom decision add`, or records that the window is refused, in which case a follow-up makes reed's revival of the orch strand drop the remote-control argument while orch.yaml's remote_control is off."
  - id: F2
    severity: LOW
    class: consistency
    origin: assist:assist
    location: _lym/landing/summary.md:19
    summary: The summary says a hub with no orch.yaml gets only the marker, but the walk seeds the absent base before the reinterpretation runs, so such a hub is flagged and its orch is mailed.
    fix: "In _lym/landing/summary.md, replace the sentence 'A hub with no orch.yaml gets only the marker, as the walk does for every absent base file.' with one saying that the walk's ReconcileHubWideAt pass (SeedDefaults) first seeds an absent orch.yaml from the prime's copy or as the header-only base, so the reinterpretation then reads that file and flags it when it sets no remote_control."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/configreg/configreg.go:161
    summary: No test fails if the orch row drops its Reinterpretations entry, so the registration that makes every hub get the notice is uncovered.
    fix: "In the configreg registry tests (the file holding TestModules_SeedOnlyAndHubWideFlags), add an assertion that configreg.Modules() returns an orch module whose Reinterpretations hold exactly one entry with ID \"orch-remote-control-off-by-default\" (a literal, not the constant) and a non-nil Rewrite."
---

### [MEDIUM] Stored resume lines keep the old default's remote control

**Location:** internal/reedengine/lifecycle.go:882

**Issue:** The board entry's first Fix bullet asks that no hub gets remote control "on any launch or resume unless its config turns it on".
`relaunchStrand` (`internal/orchcli/start.go:784`) rebuilds the spec through `launchRemoteControlName` and honours the new default.
Reed's own revival does not: `Resume` launches the strand's stored `ResumeCmd` (`internal/reedengine/lifecycle.go:882`), and so does the absent-pane relaunch (`internal/reedengine/strand.go:615`).
An orch launched by the old binary stored a resume line carrying the remote-control argument, so a revival after the upgrade starts it with remote control on although no config asks for it.
The summary names this window ("keeps it until its next launch"), but the decision record carries no added entry accepting it.
The window closes at the orch's next launch through orchcli, so the cost is bounded; whether that is acceptable is the parent's call, not the fixer's.

### [LOW] Summary misstates the absent-file case

**Location:** _lym/landing/summary.md:19

**Issue:** The summary says a hub with no orch.yaml "gets only the marker".
The walk calls `configsync.ReconcileHubWideAt` with `SeedDefaults: true` (`internal/hubreconcile/walk.go:224`) before `applyReinterpretations` (`walk.go:240`).
That pass seeds an absent per-worktree default from the prime's copy or as the header-only base (`internal/configsync/configsync.go:425-427`), and orch is a per-worktree module.
The reinterpretation then reads the seeded file and, finding no `remote_control`, flags it, so the hub's orch is mailed.
The code's behaviour is the right one; the summary's claim is wrong.

### [LOW] The orch row's registration has no test

**Location:** internal/configreg/configreg.go:161

**Issue:** `TestFlagRemoteControlDefault` pins the rewrite function and `hubreconcile`'s integration test pins the walk with a fake slack reinterpretation (`internal/hubreconcile/reinterpret_integration_test.go:85`).
No test reads the orch row's `Reinterpretations`, so removing the registration fails nothing while every hub silently loses the notice the board entry's second Fix bullet asks for.
The reach report lists `configreg.Modules` as reached by many tests, but none asserts this entry.

## Rejected

- Assistant F2's second half, that the summary wrongly says beam orch does not set the remote-control name: `lym beam start` reaches the orch launch through `orchcli.BeamUp` (`internal/beamcli/cli.go:540`), the same gated path the summary names, so the summary's claim that no other role sets the name holds.
- The assistant's rejection of the registration test (kept here as F3): the gap is a behaviour the diff adds that no test would catch breaking, which the rubric's test-economy bullet files.
- code-correctness's candidate about an explicit `remote_control: null`: the lens raised no finding and could not show the stacked load treating a null differently under the two binaries; a hand-written null on a bool key is outside what the board entry asks the reinterpretation to settle.

## Holistic notes

- Fasit: the template default (Fix bullet 1), the reinterpretation (bullet 2) and the untouched RC button (bullet 3) are built; bullet 4's check finds `launchRemoteControlName` as the one site that sets a spec's `RemoteControlName`, with reed's stored-resume replay as the exception F1 names.
- Registry contract: the ID is unique, `FlagRemoteControlDefault` returns the input bytes unchanged with a nil `rewrote`, `applyReinterpretations` then writes only the marker and mails the flagged line once, after the commit.
  Empty, null and comment-only documents decode to a nil map and are flagged; a non-mapping document errors before any write and the walk restores its snapshot.
- PATTERN-config-layers: the meaning change ships a reinterpretation with a way forward; no PATTERN entry is declared changed.
- Commits: the Root-Cause trailer names the template's `remote_control: true`, the defect the diff removes; no Loosened-Check or Pattern-Departure trailer is present or needed, as the rewritten `TestLoadConfig_RemoteControl` rows keep the empty-name refusal under an explicit `true`.
- Tests take their expected values from literals.
