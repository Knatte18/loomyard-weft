---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/sharedbootstrap.go:366
    summary: After a reboot the resume's restore takes ensureStatusStrand's replace branch, whose ReplaceStrand failure is logged and returned as nil, so the resume reports a clean success with no status_pane and the operator gets neither the pane nor the notice the board entry and doc.go promise
    fix: "In internal/loomcli/sharedbootstrap.go declare a sentinel such as `errStatusStrandNotReplaced` and, in ensureStatusStrand's replace branch, return `fmt.Errorf(\"%w: %w\", errStatusStrandNotReplaced, err)` instead of logging and returning nil. At the start call site (internal/loomcli/start.go:600), treat `errors.Is(err, errStatusStrandNotReplaced)` as today's non-fatal case: log the same Warn and carry on, refusing only on other errors. Leave the wiring.go closure returning the error unchanged, so restoreStatusPaneBeforeWake sets status_pane for a failed replace. Update ensureStatusStrand's doc comment to state that a failed replace returns the sentinel, and that every caller treats it as non-fatal."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/smoke_drivingsurface_test.go:186
    summary: No test checks that a real resume over a stale or dead status strand leaves a live one, because TestResumeVerb fakes the seam and the one wake-path smoke subtest never reads the status strand; the mutation survivor at wiring.go:1384 drops the restore and every test still passes
    fix: "In the subtest \"loom resume leaves a follow-off session off\" of TestSmokeStatusStrandAcrossDriverSeeds, record the GUID of the status strand present before the resume, which is the one the earlier subtest added with no sidecar. After the resume's existing assertions, read eng.Status(). Assert that exactly one strand matches statusStrandDisplayName, that it is Live, and that its GUID differs from the recorded one. Update the subtest's comment or the test's doc comment to say the resume replaces that strand."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/sharedbootstrap.go:1
    summary: "The file header (\"steps that start and step share\", \"ensureStatusStrand is start's alone\"), ensureStatusStrand's doc comment (\"Only start calls it\", \"ends before the run-lock probe\") and the Warn texts \"for this run\" and \"the next start will replace it\" are false now that resume calls it, after its own run-lock probe"
    fix: "In internal/loomcli/sharedbootstrap.go, rewrite the file header to say the helpers are shared by start, step and resume. Say that ensureStatusStrand is called by start and by resume's status-pane restore, never by step. In ensureStatusStrand's doc comment, replace \"Only `start` calls it, after it has brought reed up\" and the run-lock-probe sentence with a caller-neutral statement: it runs after reed is up, under the bootstrap lock its caller holds. Change the Warn at line 379 to say the next start or resume replaces the strand."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md#Design decisions
    summary: "The summary claims resume \"still never switches a viewer's window\", but the range's own Loosened-Check trailer measured the view moving from @1 to @3, because ReplaceStrand selects the new overview window when the replaced status strand's window was the viewer's"
    fix: "In _lym/landing/summary.md under Design decisions, replace \"so it still never switches a viewer's window\" with a statement that resume never calls FollowRun and leaves the viewer's window as it is, except when the viewer sits on a status strand window the restore replaces, where reed moves the view to the replacement's window."
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/resume.go:276
    summary: "The resume verb's Short, \"wake this task's parked loom driver after a halt, without starting anything\", contradicts its new Long, which brings the tmux session up and adds or replaces the status strand"
    fix: "In resumeCmd in internal/loomcli/resume.go, change Short to drop \"without starting anything\", for example \"wake this task's halted loom driver and restore its status pane\"."
---

### [MEDIUM] A failed status-strand replace on resume is silent

**Location:** `internal/loomcli/sharedbootstrap.go:366`

**Issue:** After a reboot, `reed.Up` boots the session and `clearAllPaneBindings` (`internal/reedengine/lifecycle.go:725-727`) leaves the persisted status strand not live.
`resolveStatusStrandAction` (`internal/loomcli/bootstrap.go:231-232`) then picks replace, which is the board entry's own situation.
When `ReplaceStrand` fails there, `ensureStatusStrand` logs and returns nil (`sharedbootstrap.go:366-369`).
`restoreStatusPaneBeforeWake` (`resume.go:191-201`) then returns "", and the envelope carries no `status_pane`.
The board entry's fallback, that the run says the pane is gone and how to restore it, is not met on this branch.
`doc.go:112`, the `resumeCmd` Long and the change description all claim that a failed restore is named in `status_pane`.

### [MEDIUM] No test covers the real restore seam

**Location:** `internal/loomcli/smoke_drivingsurface_test.go:186`

**Issue:** `TestResumeVerb` installs a counting fake for `restoreStatusPane` (`resume_test.go:302-307`).
The only smoke subtest that resumes on the wake path checks the follow state, the window and the spawn message, never the status strand (`smoke_drivingsurface_test.go:164-187`).
The mutation report's survivor at `wiring.go:1384` confirms that dropping the closure, which is the behaviour the board entry asks for, breaks no test.
The subtest already starts from a stale status strand, with no sidecar and added by the earlier subtest, so it can assert the replacement.

### [LOW] sharedbootstrap.go still says only start calls ensureStatusStrand

**Location:** `internal/loomcli/sharedbootstrap.go:1`

**Issue:** The file header (lines 1 and 6) and the doc comment (lines 334-336) name `start` as the only caller and place the call before the run-lock probe.
Resume now calls it through `wiring.go:1384-1389`, after its own run-lock probe at `resume.go:100`.
The Warn texts at lines 368 and 379 speak of "this run" and "the next start".

### [LOW] The change description says resume never moves the viewer's window

**Location:** `_lym/landing/summary.md#Design decisions`

**Issue:** The trailer on c4f96e75a records `current window after the resume = @3, want @1`.
The status strand is overview-anchored (`bootstrap.go:36`), and `ReplaceStrand` selects the new overview window when the replace ends the viewer's window (`reedengine/replace.go:93-95`).
The summary's "never switches a viewer's window" is therefore false.

### [NIT] resume's Short says it starts nothing

**Location:** `internal/loomcli/resume.go:276`

**Issue:** The Short still says "without starting anything", but the verb now brings the tmux session up and adds or replaces the status strand.

## Rejected

- Lens code-correctness finding 2 (a failed restore followed by a refused wake drops `paneWarning`): the restore failure is logged, and every refusal on those paths has the operator re-run `lym loom resume` or another verb that runs the restore again. The run does not drive on, so the board entry's case of a run running without its pane never arises. This is not a defect.
- Focus entry 3 (resume paths that return without a restore): a running run with a live driver or a held lock is not resumed, so it is outside the entry. A halted or running run refused for having no live driver is told `lym loom start`, which restores the pane (`start.go:595-604`).
- Focus entry 4 (Up after the driver reads): on the parked path the driver is live, so the session is warm and Up clears no bindings. On the Go-spawn path a booted Up clears only bindings of strands that are already dead, and `spawnGoDriver` reads none of them. `start` runs Up under the same bootstrap lock, so the lock window is not new.
- Focus entry 5 (no watchdog spawn on resume): the watchdog reconciles sessions and plays no part in the status pane. The board entry asks only for the pane. Resume did not spawn the watchdog before the change either.
- Focus entry 8 (the Loosened-Check trailer): it names a measured cause, the observed window ids, and the subtest still checks its name's claims, the follow state and an unchanged window. The behaviour the setup now avoids is filed against the summary's claim as F4.
