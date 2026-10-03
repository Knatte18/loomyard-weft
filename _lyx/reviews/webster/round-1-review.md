---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loomcli/selfreport.go:170
    summary: Card 10's new and changed doc comments in selfreport.go, selfreport_test.go and selfreport_github_test.go are wrapped at a fixed column, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: MEDIUM
    location: internal/loomcli/step_test.go:100
    summary: TestStepCmd_BusyRefusal_BeforeBootstrap's new marker and entryObservation assertions pass vacuously, because with no status file observeEntry returns before it consumes the marker, so an observation moved ahead of the busy probe would still pass
  - id: F3
    severity: LOW
    location: internal/loomcli/selfreport.go:289
    summary: The isPending skip in runFilingPass's detection loop is exercised by no test; a pending title whose retry fails and that the same pass detects again could be filed twice and appended to Pending twice with every test still green
  - id: F4
    severity: LOW
    location: internal/loomcli/selfreport.go:170
    summary: detectAndFileAnomalies' new paragraph is ungrammatical; the clause naming the unusable status file has no verb linking it to the filing pass
---

Review range: `git log --first-parent --no-merges -p 5ee34d63e..HEAD`.
`_lyx/shed/agent-self-reflect/status.json` gives `product.parent: main`;
`_lyx/loom/rework/round-1/record.json` carries `class: required`, `first_card: 10` (matching the plan's frontmatter) and `head_sha: 5ee34d63e…`, so the range covers the commits for cards 10, 11 and 12.

Verify commands on HEAD: `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass.

Focus items settled without a finding:

- `runFilingPass`: a title stays in at most one of `Titles` and `Pending` on every path.
  A retry success calls `settle`, which deletes the entry and records the title;
  a retry failure leaves the entry and adds nothing;
  a pending title detected again is skipped by `has` after a successful retry and by `isPending` after a failed one.
  Each `writeFiledMarker` writes the whole in-memory marker, which only accumulates within the pass, so no earlier write is lost.
  An unreadable marker reads as empty, so with no new anomaly the pass returns without overwriting it, and with one it overwrites it — the trade the plan's Shared Decision already accepts.
- `observeStepEntry` is reached only on `loomPreStepUnmarked`'s success path, after `seedAndCommitBootstrap`'s `VerifySeedOwnership`, and `observeEntry` consumes the step clean-handoff marker through `consumeStepHandoffMatch` once a status file is found.
- No `&loomCLI{...}` literal reaches `loomAfterStep` or `loomPostRun` with `cfg.Selfreport` on and a nil `fileIssue`:
  the literals in `arm_seed_test.go`, `parentreview_integration_test.go` and `wiring_commitstatus_test.go` never call either hook, `friction_test.go` and `cli_test.go` drive the hooks with the knob off, `halt_test.go`'s fixture sets a fake, and `step_test.go`'s busy refusal never reaches `AfterStep`.
  Production builds `loomCLI` only through `newLoomCLI`.
- The halt tests are not vacuous: `haltFixture` replaces `refuseFileIssue` with a recording fake, `shedverbs` calls `AfterStep` on both the error and success arms of `shed.Step`, so the second step of the same halt does run detection, and each skip case seeds a blocked status that would otherwise file.
- Every caller of the changed symbols (`runFilingPass`, `detectAndFileAnomalies`, `writeHaltNote`, `reflectHalt`, `loomPostRun`, `newLoomCLI`) is inside `internal/loomcli` and covered by the diff.

### [MEDIUM] Card 10's comments are fixed-column wrapped

**Location:** `internal/loomcli/selfreport.go:170` (`detectAndFileAnomalies`' new paragraph), and the doc comments of `detectFinalAnomalies`, `runFilingPass`, `pendingAnomaly` and `selfreportFiledMarker` in the same file;
the doc comments of every new test and of `pendHaltTitle` in `internal/loomcli/selfreport_test.go`;
the changed doc comment of `TestDetectAndFileAnomalies_FilingFailure_LeavesMarkerUnfiledAndDoesNotPropagate` in `internal/loomcli/selfreport_github_test.go`.

**Issue:** `CONSTRAINTS.md`'s Comment Line-Break Convention requires semantic line breaks in new and changed Go comments, and calls existing fixed-column wrapping legacy, not a convention to match.
These comments break mid-sentence at about column 100 (for example "returns without a write when the marker holds no pending entry and there" / "is no new anomaly.").
Card 11's comments follow the convention, so the violation is confined to card 10.

**Fix:** Rewrite each named comment one sentence per line, breaking further only at independent-clause boundaries.

### [MEDIUM] The busy-refusal observation assertions cannot fail

**Location:** `internal/loomcli/step_test.go:100`

**Issue:** Card 11 asks `TestStepCmd_BusyRefusal_BeforeBootstrap` to assert that a busy refusal leaves the step clean-handoff marker in place and `c.entryObservation` zero, pinning that the observation follows the busy probe.
The test seeds no status file, and `observeEntry` returns the zero observation at `!found` before `consumeStepHandoffMatch` runs.
An `observeStepEntry` call moved to the top of `loomPreStepUnmarked` would therefore also leave the marker and a zero observation, and the test would pass.

**Fix:** Seed a status file whose history length and state match the recorded marker, so an observation taken before the probe would consume the marker and set `Observed`.
The status file then exists by design, so prove the bootstrap did not run through the loom seed instead, which `seedAndCommitBootstrap` writes and the test never creates.

### [LOW] The isPending skip has no test

**Location:** `internal/loomcli/selfreport.go:289`

**Issue:** `TestDetectAndFileAnomalies_PendingTitleDetectedAgain_FiledOnce` covers a pending title detected again only when its retry succeeds, where `has` does the skipping.
When the retry fails, only `isPending` keeps the detection loop from filing the title a second time and appending a duplicate pending entry, and removing it leaves every test green.
That is the failure path of the "at most one of `Titles` and `Pending`" rule card 10 states.

**Fix:** Add a case where a pending title's retry fails and the same pass detects it again, asserting one filing call and one pending entry.

### [LOW] Ungrammatical sentence in detectAndFileAnomalies' doc

**Location:** `internal/loomcli/selfreport.go:170`

**Issue:** "Every path past the skips reaches the filing pass, a status file that is missing, unreadable or carries an undecodable product with no new anomalies, so …" has no verb tying the status-file clause to the pass.

**Fix:** Reword, for example: "Every path past the skips reaches the filing pass; a status file that is missing, unreadable or carries an undecodable product reaches it with no new anomalies."
