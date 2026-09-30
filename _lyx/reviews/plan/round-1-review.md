---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/03-start-strand-by-driver.md
    summary: No test pins the first acceptance criterion as a whole, that an llm-seeded start adds no status strand while its driver strand still spawns
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/00-overview.md
    summary: No card tests the failed-reed-Up acceptance criterion for start on either arm or for step's bootstrap refusal kind
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/03-start-strand-by-driver.md
    summary: The docs/overview.md edit misses the bootstrap bullet near line 473, which still says start adds the lyx loom status strand unconditionally
  - id: F4
    severity: LOW
    location: _lyx/plan/01-status-strand-removal-helper.md
    summary: The helper's tests never assert the warning the acceptance criterion requires on a failed removal
  - id: F5
    severity: LOW
    location: _lyx/plan/03-start-strand-by-driver.md
    summary: The prose sweep misses integration_driverbootstrap_test.go's header and smoke_starttail_test.go's the-status-strand-is-the-only-non-driver-strand clause
  - id: F6
    severity: NIT
    location: _lyx/plan/02-step-drops-status-strand.md
    summary: The new smoke tests are not told to call tmuxBinaryPath, so a machine without tmux fails them instead of skipping
---

The plan carries every Decision and every Constraint in the decision record, and its three cards are correctly typed and sized.
The findings are test and prose gaps against the acceptance criteria and the Documentation Lifecycle, not design defects.

Criterion-by-criterion carriage, as the focus directive asks:

- llm-seeded start over a strand-free session adds none, driver strand still spawns: only in pieces, see F1.
- llm-seeded start removes every `loom-status` strand, non-recursively, before the driver spawn: Card 3's `TestSmokeStart_LLMRunRemovesEveryStatusStrand`, plus Card 1's `recursive == false` and two-strand table cases.
- Failed llm-arm removal still reaches the driver spawn and warns: carried by construction.
  `removeStatusStrands` returns nothing, so `start` has no failure path on it, and Card 1's first-remove-fails case pins the carry-on.
  The warning itself is unpinned, see F4.
- go-seeded start keeps keep/replace/add and the sidecar write: `ensureStatusStrand`'s body stays byte-for-byte apart from `Up`, and the two `smoke_test.go` cases, now go-seeded, pin one strand after one and after two starts.
- Unseeded start adds no status strand: Card 3's edit to `TestSmokeStart_AttachTailAddsNoOperatorStrand`.
- Reed `Up` on both arms before strand work: structural in Card 3's step 4.
- Failed reed `Up` fails start on either arm, and fails step with `bootstrap`: no card tests it, see F2.
- step on both drivers, both verbs, adds no status strand: Card 2's `TestSmokeStep_AddsNoStatusStrandOnEitherDriver`.
- step leaves an existing status strand untouched: Card 2's `TestSmokeStep_LeavesAPreexistingStatusStrandUntouched`.
- Busy step refuses at the early probe: the existing `TestParity_LoomStep_RunLockBusy`, which verify runs under `-tags integration`.
- Tripwire test passes with no carve-out: `bootstrap_test.go`, in verify's plain run.
- shedcli parity tests pass and their comments are corrected: Card 2.
- CONSTRAINTS.md and docs/overview.md: Card 3, with the overview gap in F3.

Focus 2, Card 3's go-seeding of the two `smoke_test.go` bootstraps: the premise holds.
Both tests run today on an unseeded fixture that `resolveSeedDriver` seeds `llm`, so they already exercise the llm arm, and the providerless shuttle config fails its launch.
Their hard assertions are one status strand, which after this change is a go-arm property, and a seeded status file.
Their best-effort `findDriverPIDs` probe looks for a detached `lyx loom run`, which only the go arm spawns.
Seeding go does not narrow the llm path's start-level coverage.
The seed and commit block they also exercise is driver-blind, and the llm arm keeps its own start-level tests: the unseeded attach-tail case, the driver-strand reentrancy case, and the new removal case.
With F1 fixed, it also gets a case asserting no status strand beside a spawned driver strand.

Focus 3, Card 2's four step cases: each can reach and pass its assertions on the providerless fixture.
`TestSmokeStep_RecordsCleanHandoffMarkerMatchingPersistedStatus` already steps a go-seeded `newWiredPairFixture` through the real binary.
A single step dispatches only the first producer row, a pure-Go precondition row, on either seed, so the zero-real-LLM-subprocess claim holds.
`lyx shed step` defaults its run id to `self` (`Use: "step [<run-id>]"`) and reaches the same `loomPreStep`.
The "reed is up" assertion is not vacuous: `reedengine.Engine.Status` calls `requireSessionLocked` and fails when no session exists.

Focus 4, dropping `c.reed.Up()` from `ensureStatusStrand`: after Card 2, the only caller is `start`, and Card 3 puts `c.reed.Up()` directly ahead of the branch, so every remaining caller brings reed up first.
The `start.go` child-died comment near line 234 sits inside the `mustSpawn && !mustUseLLMDriverArm(driver)` block, the go path, where the status strand still exists.
It is not stale.
The same holds for `dispositionForHandshake`'s "the status strand sitting in the session" in `bootstrap.go`, since the handshake is go-only.
`integration_driverbootstrap_test.go:22` is stale, see F5.

### [MEDIUM] No test pins "adds none while the driver strand still spawns"

**Location:** `_lyx/plan/03-start-strand-by-driver.md`

**Issue:** The first acceptance criterion is that `start` on an llm-seeded run over a session with no status strand adds none, and the driver strand is spawned as today.
Card 3's attach-tail edit asserts zero status strands, but on the providerless fixture, where the driver launch fails, so the driver strand never spawns.
`TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` spawns a real (stubbed) driver strand on an llm-seeded fixture, but asserts nothing about the status strand.
No test observes both halves in one session, so a change that re-added the status strand only once a driver launch succeeds would go unseen.

**Fix:** Card 3 also edits `internal/loomcli/smoke_driverstrand_test.go`.
After each of the three bootstraps in `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps`, assert `statusStrandCount(t, eng, statusStrandDisplayName) == 0` beside the existing driver-strand count, and name the new property in the test's doc comment.

### [MEDIUM] The failed-reed-Up criterion has no test

**Location:** `_lyx/plan/00-overview.md` (Shared Decisions; no card carries it)

**Issue:** The acceptance criteria require that a failed reed `Up` fails `start` on either arm with an error envelope before the watchdog and driver spawn, and fails `step` with the bootstrap refusal kind.
The Shared Decisions state this, but no card tests it, and the decision record's scope asks for tests for all of the above.
Card 3 moves `Up` out of `ensureStatusStrand` into new code in `start`, and Card 2 replaces the call in `step`, which is exactly where a dropped `Release` or a wrong refusal kind would slip in.
A deterministic seam exists: `reedengine.Engine.Up` validates `mouse` through `mouseOption` before any tmux round trip, and `LoadConfig` does not validate it, so a fixture whose reed config has an invalid `mouse` value fails `Up` and nothing earlier.

**Fix:** Card 2 adds `TestSmokeStep_FailedReedUpRefusesWithBootstrapKind` to `smoke_drivingsurface_test.go`.
On a go-seeded fixture whose reed config replaces the template's `mouse:` line with an invalid value, `lyx loom step` exits 1 and its envelope carries `"kind": "bootstrap"`.
Card 3 adds `TestSmokeStart_FailedReedUpRefusesOnEitherDriver`, a table over go- and llm-seeded fixtures with the same reed config.
There, `lyx loom start --no-attach` exits 1 with an `ok: false` envelope, and `findWatchdogPIDs` and `findDriverPIDs` find nothing.

### [MEDIUM] docs/overview.md's bootstrap bullet stays stale

**Location:** `_lyx/plan/03-start-strand-by-driver.md` (docs/overview.md paragraph); `docs/overview.md:473`

**Issue:** Card 3 rewrites only the `start` description's second step in the loom module entry (line 340).
The overview describes `start` a second time, in its "the bootstrap" bullet near line 473: "brings up the worktree's tmux session, adds the `lyx loom status` strand (replaced in its slot when the live one was launched from a different `lyx` build), spawns the per-hub watchdog daemon ...".
After this change that bullet is wrong for every llm-driven and unseeded run, and the decision record's scope and Documentation Lifecycle constraint require the overview to reflect the new behaviour in the same commit.

**Fix:** Card 3's docs/overview.md paragraph also rewrites that bullet.
It says `start` keeps or adds the status strand, replaced in its slot on a build change, on a go-driven run, and removes any status strand on an llm-driven run.

### [LOW] The removal helper's warning is unpinned

**Location:** `_lyx/plan/01-status-strand-removal-helper.md`

**Issue:** The acceptance criterion says a failed removal proceeds and logs a warning.
Card 1's test list pins the proceeding (`status` fails and returns normally; the second remove still runs), but never the warning, so a refactor that silently swallowed the error would still pass.
The package can observe it: `logger.SetOutput` is what `internal/loomshed/gatefindings_test.go` and `internal/landingshed/publish_test.go` already use to capture log output.

**Fix:** In the `status`-fails case and the first-remove-fails case, capture the logger through `logger.SetOutput` (restored in `t.Cleanup`) and assert one warning line, naming the failing guid in the remove case.

### [LOW] The prose sweep misses two stale test-file comments

**Location:** `_lyx/plan/03-start-strand-by-driver.md`; `internal/loomcli/integration_driverbootstrap_test.go:22`; `internal/loomcli/smoke_starttail_test.go:4`

**Issue:** `integration_driverbootstrap_test.go`'s header justifies driving `startLLMDriverArm` directly by saying "loom start" also spawns the status strand's own watcher pane via `os.Executable()`.
That test is llm-seeded, and after Card 3 an llm-seeded start spawns no status strand, so the stated hazard no longer applies to its own case.
`smoke_starttail_test.go`'s header says the attach tail adds no loom-operator strand, "so the status strand is the only non-driver strand".
Card 3 changes that very test to expect zero status strands, but its instruction to "correct the header comment the same way" names only the ensureStatusStrand-needs-tmux clause.

**Fix:** Card 3 edits `integration_driverbootstrap_test.go`'s header so the watcher-pane hazard is the go-driven arm's, and keeps the rest of its rationale.
Card 3 names the `smoke_starttail_test.go` header clause explicitly.
On the unseeded, llm-seeded fixture the attach tail adds neither a loom-operator strand nor a status strand.

### [NIT] New smoke tests do not skip without tmux

**Location:** `_lyx/plan/02-step-drops-status-strand.md`; `_lyx/plan/03-start-strand-by-driver.md`

**Issue:** Every new test brings reed up and asserts through `probeReedEngine(...).Status()`, which needs a tmux binary.
The cards never say to call `tmuxBinaryPath(t)` first, which is how this suite's live-tmux tests skip on a machine without tmux.

**Fix:** Each new test in `smoke_drivingsurface_test.go` that needs a live session starts with `tmuxBinaryPath(t)`.
F2's failed-Up tests need no tmux and are exempt.
