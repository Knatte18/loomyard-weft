---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loomcli/bootstrap.go:245
    summary: removeStatusStrands passes printf formats to the slog-style logger.Warn, so the warnings carry literal %s/%v verbs and malformed attrs (the guid becomes an attr key, the status error a !BADKEY value)
  - id: F2
    severity: MEDIUM
    location: internal/loomcli/bootstrap_test.go:838
    summary: TestRemoveStatusStrands pins the failing guid with wantInLog a, which the message text itself already contains, so the check is vacuous and let F1 through
  - id: F3
    severity: MEDIUM
    location: internal/loomcli/bootstrap.go:233
    summary: New and changed Go comments across the diff are fixed-column wrapped with mid-phrase breaks and several sentences per line, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F4
    severity: NIT
    location: internal/loomcli/arm.go:390
    summary: The reworded early-probe comment lost its conjunction and now reads step would seed, commit, bring up reed against a task
  - id: F5
    severity: NIT
    location: internal/loomcli/start_watchdog_test.go:6
    summary: The reworded start_watchdog_test.go header says reed Up and the strand branch both call c.reed.Up() and c.reed.Status(), which is inaccurate for either one taken alone
  - id: F6
    severity: NIT
    location: internal/loomcli/smoke_drivingsurface_test.go:28
    summary: stepSmokeTimeout is documented as bounding one step invocation but also bounds the three start invocations in the same file
---

Range: `product.parent` is `main` (from `_lyx/shed/status-strand-by-driver/status.json`); reviewed `git diff $(git merge-base main HEAD)..HEAD`, the three card commits.

Evidence gathered: `go build ./...`, the plan's unit and integration test commands, and `go vet -tags smoke` all pass.
Beyond the plan's verify block, I ran the smoke tier for every new and changed smoke test against a live tmux server.
All pass; `TestSmokeBootstrap_MalformedStatusProceedsToHandoverAndLogsWhy` and `TestSmokeBootstrap_DiedDriverProceedsToHandoverAndLogsWhy` fail only when `$TMUX` is set in the test's environment (the handover takes the inside-another-tmux branch), and pass with it unset — an environment artifact of running inside a reed pane, not a defect of this diff.

### [MEDIUM] removeStatusStrands formats its warnings printf-style for a key/value logger

**Location:** internal/loomcli/bootstrap.go:245 and :253
**Issue:** `logger.Warn(msg string, args ...any)` forwards to `slog.Logger.Warn`, which treats `args` as key/value pairs; every other `internal/loomcli` call site passes `"key", value` pairs.
`logger.Warn("remove status strand %s: %v", s.GUID, err)` logs the literal message `remove status strand %s: %v` with an attribute whose key is the guid and whose value is the error (e.g. `a=refused`).
`logger.Warn("read strand table to remove status strand: %v", err)` logs the literal `%v` and `!BADKEY=boom`.
Card 1 asks for a warning naming the failing guid and the error; the guid only appears as an attribute key, which is not a readable warning.
**Fix:** Use the package's key/value style: `logger.Warn("loomcli: could not remove a status strand; leaving it up", "guid", s.GUID, "cause", err)` and `logger.Warn("loomcli: could not read the strand table to remove the status strand; leaving any in place", "cause", err)`.

### [MEDIUM] The "first remove fails" case does not pin the guid it claims to pin

**Location:** internal/loomcli/bootstrap_test.go:838 (the `first remove fails` row) and the `wantInLog` assertion
**Issue:** `wantInLog: "a"` is a single letter that the message text (`strand`, `status`, `trace=`) already contains, so the assertion passes whatever the warning says.
Card 1 requires the warning naming the failing guid to be "pinned rather than assumed"; this pin is exactly what let F1 through.
The line-count check itself does pin exactly one warning in both failure cases (slog's text handler writes one line per record), so that half is sound.
**Fix:** Give the strands distinguishable guids (e.g. `guid-a`, `guid-b`), assert the log contains `guid=guid-a`, and assert it does not contain the succeeding guid.

### [MEDIUM] New and changed comments break CONSTRAINTS.md's Comment Line-Break Convention

**Location:** internal/loomcli/bootstrap.go:233 (removeStatusStrands doc), and likewise every new or changed comment in the diff
**Issue:** CONSTRAINTS.md requires semantic line breaks in new and changed Go comments — one sentence per line, a further break only at an independent-clause boundary.
The diff's new and reworded comments are fixed-column wrapped, breaking mid-phrase (`for the llm arm, where the` / `ly-drive strand is ...`), and several put two sentences on one line (sharedbootstrap.go's header: `... releases before calling the producer. ensureStatusStrand is ...`).
Affected: `removeStatusStrands`'s doc; arm.go's `loomPreStep` doc and early-probe comment; sharedbootstrap.go's file header sentences and `ensureStatusStrand`'s first doc sentence; start.go's file header sentence, `mustUseLLMDriverArm` doc, the RunE driver comment's reworded sentence, the step-4 comment, the llm-arm inline comment and the watchdog-placement comment's second point; the smoke_drivingsurface_test.go file header and its doc comments; and the reworded sentences in integration_driverbootstrap_test.go, start_watchdog_test.go, smoke_starttail_test.go, smoke_driverstrand_test.go and shedcli/parity_test.go.
**Fix:** Re-break each touched sentence at sentence and independent-clause boundaries, leaving untouched legacy lines as they are.

### [NIT] Early-probe comment reads ungrammatically after the rewording

**Location:** internal/loomcli/arm.go:390
**Issue:** "step would seed, commit, bring up reed against a task a live driver owns" dropped the "and" when the strand clause went.
**Fix:** "step would seed, commit and bring up reed against a task a live driver owns."

### [NIT] start_watchdog_test.go header misstates which call makes which reed call

**Location:** internal/loomcli/start_watchdog_test.go:6
**Issue:** "both call c.reed.Up() and c.reed.Status()" — `c.reed.Up()` does not call itself, and the strand branch calls `Status` but not `Up`.
**Fix:** Say that `c.reed.Up()` and the strand branch's `c.reed.Status()` run on a concrete `*reedengine.Engine`.

### [NIT] stepSmokeTimeout also bounds start invocations

**Location:** internal/loomcli/smoke_drivingsurface_test.go:28
**Issue:** The constant's doc says it bounds one step invocation and argues from step spawning no driver, but `TestSmokeStart_LLMRunRemovesEveryStatusStrand` and `TestSmokeStart_FailedReedUpRefusesOnEitherDriver` use it for `start`.
**Fix:** Rename it to cover both verbs (e.g. `verbSmokeTimeout`) and document it as bounding one step or `start --no-attach` whose producer or driver launch fails fast on the providerless shuttle config.

## Focus directives, settled

- **logger.Warn format** — defect, F1.
- **wantInLog "a" and the line count** — the guid pin is vacuous (F2); the line count does pin exactly one warning in both failure cases.
- **removeStatusStrands vs reedDriverPaneProbe** — not a defect.
  `reedDriverPaneProbe` is an adapter to the `driverPaneProbe` interface whose one removal method is named and documented for the driver strand; `removeStatusStrands` adds the name filter, the every-match loop and the warn-and-continue stance, and Card 1 prescribes exactly this function-value shape.
  Reusing the probe would route status-strand removal through `RemoveDriverStrand`, a misleading name for no saved logic.
- **Comment convention** — defect, F3, covering every listed site.
- **Four extra seedGoDriverRun calls in smoke_test.go** — not a defect.
  Each of the four asserts a detached Go driver property: `RunStandalone`, `MalformedStatus` and `DiedDriver` kill and read `findDriverPIDs`/`LoomDriverLog`, and `ConcurrentSpawn` samples `findDriverPIDs`.
  Since commit 290908983 an unseeded run seeds `llm`, whose arm spawns no detached `loom run`, so without the seed those tests would pin nothing about a Go driver (or fail on a missing driver log); all four pass with it.
- **Smoke tier traceability** — every test in smoke_drivingsurface_test.go and every changed smoke assertion passes against live tmux: the envelope loop finds the JSON line among stderr's slog lines, `output.ErrFields` writes compact JSON so `"ok":false` matches, `newBadReedUpFixture` mirrors `newWiredPairFixture`'s hub build, and `mouseOption` fails in `ensureServerAndSessionLocked`'s pre-tmux validation block.
  `stepSmokeTimeout` suffices for `start` (the providerless startup window is 2s); only its naming is off (F6).
- **Lock and failure contracts** — kept.
  Start: a failed `Up` releases the bootstrap lock and writes the error envelope; the llm arm never fails; the go arm keeps `ensureStatusStrand`'s release-and-envelope path; the watchdog spawn sits after the branch on both arms.
  Step: a failed `Up` releases the lock and returns `KindBootstrap`, and step never reaches the watchdog.
  `ensureStatusStrand` has one caller left (start's go arm), which calls `Up` first, so nothing relied on its removed `Up`.

Per-card mechanical checks: no card carries a Rename or Delete group; Card 1's Create target has no prior equivalent (settled above), and Card 2's created file exists with the three named tests.
