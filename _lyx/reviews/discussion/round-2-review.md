---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D4 names shuttle's autonomous-posture denies as a bound on a batten tackler's prime writes, but no shuttle deny covers file writes, git add or git commit, so the only bound is unenforced prompt text while a resume can re-enter Land and push the tackler's commit
    fix: In D4, delete the clause naming shuttle's autonomous-posture denies from the sentence at line 87 and from the Bound line at line 92, and state that the stencil's write rule is the only bound and that it is not enforced. Then either add a mechanical bound to D4 that holds despite the prime's other writers, with a matching acceptance criterion (for example, batten refuses a tackler resume whose goto is Land or Worktree-Teardown and hands back instead), or keep the prompt-only bound and record under Open risks that a batten tackler's local commit or index change in the prime can be carried into Land's push.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:72
    summary: "D3 says no run has driver records today, but llm-seeded runs already hold drive reports under shedrun.DriveReportsDir, which the LLM driver writes and reportengine reads, and several listed records are already declared in shedrun"
    fix: In D3, split the record list at line 72. Name the records shedrun already declares and whose paths stay unchanged (drive reports via DriveReportsDir, the driver lock, the park marker, the tackler directory and marker), and name only the Go-driver-only records that change home (the settle records, now under loom's friction directory via SettleRecordsFile). Restate the premise at line 74 so it covers only those records, for example that no loom or darn run is seeded go, so no run holds settle records or a driver log at the old location.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D4 says a new-build batten run adopts a step the old-build process still runs through the loop waiter, but the old process runs shedverbs' generic run body holding the run lock with no loop pid file, so the new driver meets a busy stop and exits rather than adopting
    fix: Replace the sentence at line 83 with one saying that a new-build lyx batten run started while an old-build batten process is alive meets that process's run lock as a busy stop and exits, leaving the old process to finish on its own build, and that a rerun after the old process has exited resumes at the recorded row. Add an acceptance criterion for the busy exit.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:107
    summary: The loom.yaml migration replaces only driver_kind's comment, so the hub file keeps the old comments on tackler, tackler_timeout_min and tackler_max_spawns, which describe the built-in fallbacks rather than inheriting shed.yaml
    fix: In D5 at line 107, have the migration replace the comments of all four keys with the new template's comments, and extend the migration acceptance criterion at line 178 to check that all four keys carry the new template's comments.
---

### [BLOCKING] Batten tackler write bound names denies that bound no writes

**Location:** `_lyx/discussion/decision-record.md:87`, `:92`

**Issue:** D4 says what the removed tamper snapshot now lets through: a tackler that writes files, the index or a local commit in the prime has its verdict acted on, and a resume can re-enter Land or Worktree-Teardown.
That part answers the focus question.
The bound it gives is "the stencil's write rule … plus the denies of shuttle's autonomous posture".
`internal/loomcli/tacklerprompt.go:88` says the autonomous posture "adds the operator-prompt deny", which is the `AskUserQuestion` deny in `internal/shuttleengine/claudeengine/settings.go:35`.
Shuttle's other standing denies cover the Agent tool, python, machine sleep, in-place `sed`/`perl`, computed-path `rm` and module-wide `go test`, as listed in `internal/shuttleengine/claudeengine/doc.go:157-182`.
None of them stops Edit, Write, `git add` or `git commit`.
So the only bound is the line in `contracts/stencils/loom/loom-template-tackler.md:38`, which tells the tackler what to write and enforces nothing.
Acceptance criterion line 182 checks only that the prompt renders that rule.
Batten's Land row lands into the parent and the hub then deploys, so a resume into Land can carry a tackler's stray commit into the push.
Under the rubric's attack-surface rule, this guard removal has a stated bound that rests on a false premise, and the plan writer would build no mechanical check because it believes one already exists.
Round 1's fixer report says `tacklerSpec` "sets no write restriction beyond the autonomous posture, so the stated bound is the real one". That holds only if the posture restricts writes, and it does not.

### [MEDIUM] D3's no-driver-records premise is false for drive reports

**Location:** `_lyx/discussion/decision-record.md:72`, `:74`

**Issue:** D3 lists the driver lock, park marker, resume report, drive reports, settle records, driver log and tackler attempt directory as records to declare in `internal/shedrun`.
It then says "No run has driver records today … so the records move to `shedrun` paths without a migration."
Drive reports are written by the LLM driver too.
`internal/loomcli/driverreport.go:22` composes them under `shedrun.DriveReportsDir`, and `internal/reportengine/rounds.go:26` reads them.
Live llm-seeded runs hold them, for example `twill-fork0-weft/_lyx/shed/twill-fork0/drive-reports/drive-report-20261014-204011-22de.md`.
`shedrun/paths.go` already declares `DriverLock`, `TacklerDir`, `TacklerAttemptDir`, `TacklerMarker` and the park marker.
The only listed record still placed outside a run directory is `SettleRecordsFile`, which sits in the told friction directory.
A plan writer reading "the records move" could relocate drive reports, orphaning existing reports and the LLM driver's commit-records park path, though Constraints says "The LLM driver keeps working for loom and darn".
The batten half of the premise holds: the prime's batten run directories, such as `loomyard/_lyx/shed/batcher-v2/`, hold only `seed.json` and `status.json`.

### [LOW] Old-build batten adoption premise

**Location:** `_lyx/discussion/decision-record.md:83`

**Issue:** Today `lyx batten run` runs shedverbs' generic `run` body and holds the run lock throughout (`internal/battencli/arm.go:380`, "another batten run already holds the run lock").
The loop waiter adopts only a detached `--until-stop` loop through its loop lock and pid file (`internal/shedverbs/doc.go:55-70`).
An old-build batten process writes neither, so a new-build run over it does not adopt anything.
It takes the driver lock, which the old process never held, steps, meets the run lock as `KindBusy`, and `decideDrive` (`internal/loomcli/driverules.go:201`) exits it.
The outcome is safe, but D4 describes a different mechanism, and no acceptance criterion pins it.
This bites in practice: batten's Land row deploys the new build while the old-build batten process for the same landing is still running.

### [LOW] Migration leaves three stale key comments

**Location:** `_lyx/discussion/decision-record.md:107`

**Issue:** The hub's `_board/_lyx/config/loom.yaml:37-39` carries end-of-line comments "empty takes haiku", "0 takes 10" and "0 turns the tackler off".
D5's migration rewrites the four values to the inherit value but replaces only `driver_kind`'s comment.
Reconcile does not refresh an existing key's comment (`internal/configsync/configsync.go:359-381` migrates, then merges missing keys).
After migration, the operator's file says an empty `tackler` takes haiku when it in fact inherits `shed.yaml`.

## Focus verdicts not raised as findings

- Loom Migrate rewrite: D5 bounds the rewrite to a one-time hub-level match on all four old values plus the old `driver_kind` comment.
  `PrepareRunSettings` (`internal/configreg/runsettings.go:56-61`) marshals JSON with no comments before calling `Migrate`, so a `run_settings` pin is untouched, as D5 says.
  A pin set after migration is kept, and line 109 covers the inherit value in the loom, darn and batten templates.
  Not a defect beyond F4.
- Per-recipe tackler settings source: line 104 names the run's own recipe override for every tackler key and ties `lyx loom drive`'s pick to the seed's recipe, matching D6.
  Today's `goDriverDepsFor` resolves from loom's config (`internal/loomcli/drivewiring.go:47`), and the Decision states that change.
  Not a defect.
- Batten resume goto restriction: D2's resume-goto seam names it, D7 renders it, and acceptance lines 180 and 182 pin the refusal and the prompt.
  It replaces the current-or-routed-from check at `internal/loomcli/tackler.go:371`.
  Not a defect.
- D2's Bound line against `decideDrive`: an unwired verify target, watched read or hand-over leaves its row unfired, and the stop falls to the later rows, ending at the `state == blocked` tackler row or the final hand-back.
  An unwired arbiter turns the arbiter rows (`EscalationPending`, `ResumeReturned`) into hand-backs, which D2's arbiter bullet states.
  An unnamed interrupt policy reads `handback` (`driverules.go:229`).
  The Bound line matches.
- Batten in flight across the deploy: covered, but by a different mechanism than D4 states; see F3.
