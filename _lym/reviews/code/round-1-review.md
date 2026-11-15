---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: cmd/lym/testonlycode_linux_test.go:51-60
    summary: The guard allowlist keeps the pure recipe checker (shedcheck.Check, its seven helpers, Finding.String) and shedbuild.Check in production files, though none reads package state or writes a record, which PATTERN-test-only-code and the goal forbid
    fix: >-
      Move the internal/shedcheck package (Check, its helpers, Kind, Finding and Finding.String, with check_test.go, finding_test.go and testmain_test.go) into a kit under internal/testkit, for example internal/testkit/shedcheckkit.
      Delete internal/shedbuild/check.go (shedbuild.Check) and point its test callers in internal/loomrecipe and internal/shedbuild at the kit's Check directly.
      Remove the ten entries at cmd/lym/testonlycode_linux_test.go:51-60.
      Update the shedbuild comments that name shedcheck as routing's owner (build.go:30, doc.go:7, parse.go:3 and :37, recipe.go:14).
      In _lym/landing/summary.md, take shedcheck and shedbuild.Check out of the allowlist list and the claim that each kept entry reads its package's own state or writes its record.
      If PATTERN-test-isolation bars a kit from importing internal/shedengine, make no move and report that to the run's parent instead.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: pattern/PATTERN-test-only-code.md:7-17
    summary: The landed rule is wider than the board entry's rule, which allows only _test.go files and internal/testkit, and neither widening is recorded with the parent
    fix: >-
      Leave the code in place; this finding is deferred to the run's parent.
      The parent records with `lym loom decision add` whether gitkit and hubforge count as test-support packages beside internal/testkit.
      The parent also records whether a function only other packages' tests call may stay in its production file under the guard's allowlist when it reads its package's own state or writes a record its package alone writes,
      or instead directs how those functions move.
      The fixer changes no file for this finding.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/destroy.go:291-297
    summary: The live board-ownership predicate behind the Bolt replay (replay.go:755) no longer has a test of its foreign-board refusal, because the diff deleted TestBoardDropRequest_RefusesForeignBoard with dropSeedCommits
    fix: >-
      Add a package-fabricengine test, for example in internal/fabricengine/destroy_test.go, that builds a git worktree named BoardDirName under a temp directory no hub weft registers, as the deleted test did,
      and asserts that checkPathRequest(replayRequest(foreign, ownedBoardWorktree(), nil)) returns a refusal whose Check is CheckOwnership.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/export_test.go:778-833
    summary: PushAnchored has no production caller and is neither a reference nor an oracle, yet it is kept as a full test-file copy whose own tests assert its behaviour, and the change description calls it a setup helper
    fix: >-
      Delete PushAnchored from internal/fabricengine/export_test.go.
      Re-target its callers at fabricengine.PushPairAnchored, on the prime where a records-only push is meant.
      The callers are TestPushAnchored_PushesAndRecordsBranchPush and TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejected, the calls at pushanchored_integration_test.go:249 and :436, and the calls at pushclasses_integration_test.go:626, :681, :709, :732 and :753.
      Drop any row that TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide already covers.
      Update the header of pushanchored_integration_test.go (lines 1-2) and the TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks doc so neither names PushAnchored.
      Take PushAnchored out of the Setup helpers list in _lym/landing/summary.md.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/shed-recipe-spec.md:243
    summary: The shed recipe spec still says shedbuild.Load stays exported and covered for a non-embedded consumer, and the diff deletes it with its test
    fix: >-
      In contracts/specs/shed-recipe-spec.md, cut ", never `shedbuild.Load`" from line 243 and delete the Unused entry paragraph at lines 254-256.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:386
    summary: The package doc still names a Fabric.snapshotWarpSHA method that no longer exists and says a consumer reads the SHA through it, though only the test function SnapshotWarpSHAForTest remains and nothing in production reads the SHA
    fix: >-
      In internal/fabricengine/doc.go, make line 386 name `SnapshotWarpSHAForTest(f, tag)` in export_test.go as the read half the tests use.
      Rewrite lines 418-421 so they no longer describe a consumer of the read.
      Replace `snapshotWarpSHA` at lines 396, 429, 480 and 491 with the test reader's name.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/snapshot_integration_test.go:47
    summary: TestSnapshotWarpSHA_UnbornWeftHEAD, _PerBranchScoping, _TopologicalOrderBeatsCommitDate and _DanglingWarpSHA mainly assert the tag-match loop of SnapshotWarpSHAForTest, which is now test code
    fix: >-
      Delete the four TestSnapshotWarpSHA_* tests in internal/fabricengine/snapshot_integration_test.go,
      and keep SnapshotWarpSHAForTest only as the oracle at commit_integration_test.go:263.
      If one of the four pins an ordering or scoping behaviour of scanWarpSHATrailers that no RebuildIndex or WeftSHAForWarpSHA test covers,
      re-target that one test at RebuildIndex instead of deleting it.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/discussionparser/append.go:228
    summary: CheckAddedDecisionsKept's doc says AppendDecision replaces the bullet, but AppendDecision is now a test helper and the production writer is AppendDecisions
    fix: >-
      In internal/discussionparser/append.go:228, replace "AppendDecision replaces" with "AppendDecisions replaces".
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:768
    summary: The overview says planglyph.ValidateFormat and Validate call planparser's validators, but those package-level functions now exist only in planglyph_test.go
    fix: >-
      In docs/overview.md:768, replace "`planglyph.ValidateFormat`/`Validate` call `planparser.ValidateFormat`/`Validate`" with "planglyph's plan gates call `planparser.ValidateFormat`/`Validate`".
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: cmd/lym/exitcode_test.go
    summary: The contained tests the redundancy report lists each pin an assertion their covering test lacks, and none carries a testtiming keep marker
    fix: >-
      Add `//testtiming:keep <reason>` directly above the func line of each test below, naming what it pins.
      TestExitCode_HelpPaths: exit 0 with no error envelope on help paths.
      TestExitCode_UnknownModule: the unknown-command error text.
      TestExitCode_HandlerFailure: a handler failure's envelope decoding whole.
      TestJSONHelp_Schema (cmd/lym/jsonhelp_test.go): the JSON help schema.
      TestRunCLI_GroupGuard_NoGitRepoNeeded (internal/battencli/cli_test.go) and TestRunCLI_GroupGuard_OutsideGitRepo (internal/burlercli/cli_test.go): a bare group exiting 0 outside a repository.
      TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding (internal/dobbyengine/landnotice_creel_test.go): delivery through a real creel store to an orch listed after the landing.
      TestRunCLI_CloneOfAnUnmigratedWeftLeavesTheMarkerToTheWalk (internal/fabriccli/cli_test.go): the tracked marker, the walk commit's subject and the stripped fabric.yaml.
      TestRunCLI_ReconcileBacksFillsWarpBinding (internal/fabriccli/cli_test.go): the recorded warp_binding and the binding file tracked again.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/hibernatedaemon_test.go:24-33
    summary: The terminal phases now come from a hand-written list of every phase rather than from the engine, so a terminal phase the engine adds later escapes the ended-phases check
    fix: >-
      Restore hibernateengine.TerminalPhases (a clone of the package's terminalPhases) in internal/hibernateengine/request.go,
      with an allowlist entry in cmd/lym/testonlycode_linux_test.go saying it reads the package's terminal phase list for internal/fabriccli's hibernate daemon test.
      Call it again in TestHibernateEndedPhases_AreTheEnginesTerminalPhases instead of the hand-written list.
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: cmd/lym/main.go:75-86
    summary: No test pins the stdout and stderr split that run now carries, because every caller passes one buffer as both writers
    fix: >-
      In TestExitCode_UnknownModule (cmd/lym/exitcode_test.go), call run with two separate buffers for one row,
      and assert that the JSON error envelope is on the stdout buffer and absent from the stderr buffer.
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: cmd/lym/testonlycode_linux_test.go:184-206
    summary: packageFunctions skips generic functions and the methods of generic types in test-support packages, so a production function only a generic kit helper calls reads as unreachable, contrary to the pattern's every-function-is-a-root rule
    fix: >-
      Add a bound to pattern/PATTERN-test-only-code.md's Enforcement section, and a line to the guard's packageFunctions doc,
      saying that a generic function of a test-support package is not a root, so a production function only such a function calls takes an allowlist entry.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: pattern/PATTERN-test-only-code.md:24-37
    summary: The guard skips generated files and unexported empty marker methods, which the pattern's Enforcement section does not state, and the third bound says a package no main imports is outside the analysis when packages only test-support packages import are inside it
    fix: >-
      In pattern/PATTERN-test-only-code.md's Enforcement section, add that the guard skips generated files and unexported marker methods with no parameters, results or body.
      Change the third bound to "a package that neither a main nor a test-support package imports is outside the analysis".
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/configcli/configcli_test.go:191
    summary: Test names across the cli packages keep the removed RunCLI and RunCLIIn seam names, such as TestRunCLIIn_FromNonGitDirectory and TestRunCLI_GroupGuard_NoGitRepoNeeded
    fix: >-
      Rename every func TestRunCLI* and TestRunCLIIn* under internal/*cli so the name says what it drives, for example TestCommand_FromNonGitDirectory.
      Update any testtiming or redundancy reference to the old names in the same files.
  - id: F16
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/preflightshed/preflight_test.go:28-49
    summary: TestNewPreflight_CarriesToldName names the test helper NewPreflight as its subject, while what it pins is the field assignment of the live NewPreflightWithChecks
    fix: >-
      Rename the test to TestNewPreflightWithChecks_CarriesToldName and make it call NewPreflightWithChecks("Some-Row", "/some/cwd", Checks{}) directly.
      Update its doc and keep reason to name NewPreflightWithChecks.
  - id: F17
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: docs/benchmarks/board-performance.md:93
    summary: The benchmark doc says the CLI bench rows drive boardcli.RunCLI, which the diff deleted, and bench_cli_test.go now calls clihelp.Execute
    fix: >-
      In docs/benchmarks/board-performance.md:93, replace "they drive `boardcli.RunCLI`" with "they drive `clihelp.Execute(boardcli.Command(), ...)`".
  - id: F18
    severity: NIT
    class: consistency
    origin: handler
    location: cmd/lym/testonlycode_linux_test.go:67
    summary: The seatengine.Prompts allowlist reason names only darnengine's and landingshed's tests, but burlerengine, driftengine, loomcli and loomengine tests use it too
    fix: >-
      Change the Why of the internal/seatengine.Prompts entry to "oracle of the seat prompts for the tests of the packages that start seats".
  - id: F19
    severity: NIT
    class: consistency
    origin: handler
    location: internal/reedengine/proctree_windows.go:69-70
    summary: tmuxProcessName's comment explains the manual separator split by a test host that is not Windows, but the file now compiles on Windows only
    fix: >-
      In internal/reedengine/proctree_windows.go, replace the manual LastIndexAny split with filepath.Base, which handles both separators on Windows.
      Delete the two comment lines above it.
---

### [BLOCKING] The allowlist keeps the pure recipe checker in production files

**Location:** `cmd/lym/testonlycode_linux_test.go:51-60`

**Issue:** The board entry's rule is that a function only tests call lives in a `_test.go` file or in `internal/testkit`.
`PATTERN-test-only-code` lets such a function stay in a production file only when no kit can hold it, because it reads its package's own state or writes a record its package alone writes (`pattern/PATTERN-test-only-code.md:15-17`).
The guard's own comment repeats that condition (`cmd/lym/testonlycode_linux_test.go:43-45`).

`shedcheck.Check` and its helpers (`internal/shedcheck/check.go:28-265`) and `Finding.String` (`finding.go:37`) are pure functions over their arguments.
`shedbuild.Check` (`internal/shedbuild/check.go:21-23`) is a one-line call into `shedcheck.Check`.
None of them reads package state or writes a record, and the only production caller of `shedcheck.Check` is the allowlisted `shedbuild.Check`.
So the whole of `shedcheck` is test-only code in a production package.
The change description admits a kit could hold these ("Moving the pure ones, such as the recipe checkers, into a kit is possible follow-up work").
It also states, contrary to that, that each allowlisted entry reads its package's state or writes its record.

### [MEDIUM] The landed rule widens the board entry's rule without a recorded parent decision

**Location:** `pattern/PATTERN-test-only-code.md:7-17`

**Issue:** Item 4 of the board entry says a function only tests call lives "in a `_test.go` file or in `internal/testkit`, never in a production file".
The diff widens that in two ways.
It counts `internal/gitkit` and `internal/hubforge` as test-support packages.
It also keeps functions only other packages' tests call in production files under an allowlist.
Item 2 offers only deletion, or a move into a `_test.go` file for a reference or oracle.
The change description argues for both widenings as the writer's own design decisions.
The decision record has no `Added after Discussion` entry for either, and no commit carries a `Pattern-Departure:` trailer.
Whether either widening stands is the parent's call.

### [MEDIUM] The live board-ownership refusal lost its only test

**Location:** `internal/fabricengine/destroy.go:291-297`

**Issue:** `ownedBoardWorktree` still guards the Bolt replay's reset (`internal/fabricengine/replay.go:755`), through the `pathOwnershipBoardWorktree` branch at `destroy.go:559`.
The deleted `TestBoardDropRequest_RefusesForeignBoard` was the one test that pinned its refusal of a board-named directory no weft repo of the hub registers.
No `_test.go` file references `ownedBoardWorktree`, `pathOwnershipBoardWorktree` or a foreign board any more.
The focus asks that no deleted test leave a live function's behaviour uncovered.

### [MEDIUM] PushAnchored is kept as a tested test-file copy

**Location:** `internal/fabricengine/export_test.go:778-833`

**Issue:** `PushAnchored` has no production caller.
The diff moved its whole body, including the lock, the owed-side map and `coalesceFlagged`, into `export_test.go`.
`TestPushAnchored_PushesAndRecordsBranchPush`, `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejected` and several `pushclasses` rows now assert the behaviour of that test code.
Board entry item 2 deletes such a function together with the tests that test only it, and keeps one only as a reference or oracle; `PushAnchored` is neither.
The cross-package callers were already switched to the live `PushPairAnchored` on the prime, which is the same move these tests need.
The change description lists `PushAnchored` under "Setup helpers", which the tests that drive it as their subject contradict.

### [MEDIUM] The shed recipe spec still promises shedbuild.Load

**Location:** `contracts/specs/shed-recipe-spec.md:243`

**Issue:** Line 243 reads "never `shedbuild.Load`".
The "Unused entry" paragraph at lines 254-256 says `shedbuild.Load` "stays exported and covered as the entry a non-embedded consumer … would use".
The diff deletes `Load` from `internal/shedbuild/parse.go` and deletes its test, so the spec now contradicts the code.

### [LOW] fabricengine's doc names a snapshot reader method that is gone

**Location:** `internal/fabricengine/doc.go:386`

**Issue:** Line 386 names `Fabric.snapshotWarpSHA(tag string)`, a method deleted along with `snapshot.go`.
Line 387 adds that it is defined in `export_test.go`, but that file holds the free function `SnapshotWarpSHAForTest`.
Lines 418-421 still say "A consumer reads the SHA via `snapshotWarpSHA`", but no production code reads it.
Lines 396, 429, 480 and 491 also keep the old name.

### [LOW] The TestSnapshotWarpSHA tests mostly test test code

**Location:** `internal/fabricengine/snapshot_integration_test.go:47`

**Issue:** The four `TestSnapshotWarpSHA_*` tests (lines 47, 73, 169 and 328) assert the tag-match and selection loop that now lives only in `SnapshotWarpSHAForTest` (`export_test.go:445`).
That function serves as an oracle over live writes only at `commit_integration_test.go:263`.

### [LOW] CheckAddedDecisionsKept's doc names the test helper

**Location:** `internal/discussionparser/append.go:228`

**Issue:** The doc says "AppendDecision replaces the bullet it finds for an ID".
`AppendDecision` now lives in `append_test.go`, and the production writer is `AppendDecisions`.

### [LOW] The overview names planglyph wrappers that are test-only

**Location:** `docs/overview.md:768`

**Issue:** The line says "`planglyph.ValidateFormat`/`Validate` call `planparser.ValidateFormat`/`Validate`".
Those package-level functions now exist only in `internal/planglyph/planglyph_test.go`.

### [LOW] Contained tests lack keep markers

**Location:** `cmd/lym/exitcode_test.go`

**Issue:** The redundancy report lists several touched tests as contained.
These include `TestExitCode_*` and `TestJSONHelp_Schema` (covered by `TestRunDispatchesToUninitializedRepoModules`), the battencli and burlercli group-guard tests, dobbyengine's creel land-notice test, and two fabriccli tests.
Their assertions are an exit 0 on help, the unknown-command text, a whole envelope decode, the JSON help schema, a bare group exiting 0 outside a repository, real creel delivery, and the tracked marker and binding.
The covering tests assert none of these, so each test stays, and needs the marker naming what it pins.

### [LOW] The hibernate phase test no longer reads the engine's terminal phases

**Location:** `internal/fabriccli/hibernatedaemon_test.go:24-33`

**Issue:** With `TerminalPhases` deleted, the test lists all eight phases by hand and filters them through `Request.Terminal()`.
A terminal phase the engine adds later and the list omits is never checked against the daemon's ended phases.
The check that took its list from the engine is therefore weaker.
The function reads its package's own state, so the pattern admits it under an allowlist entry.

### [LOW] run's stdout and stderr split is untested

**Location:** `cmd/lym/main.go:75-86`

**Issue:** `run` now takes separate stdout and stderr writers, and its doc says the JSON error envelope goes to stdout.
Every caller passes one buffer as both writers, so no row would fail if the two writers were swapped.

### [LOW] Generic test-support functions are not roots

**Location:** `cmd/lym/testonlycode_linux_test.go:184-206`

**Issue:** `packageFunctions` skips functions and methods with type parameters (lines 189 and 194).
The guard loads no test files, so a generic kit helper that only tests instantiate has no instance in the program.
A production function only that helper calls would then fail the guard, against the pattern's rule that every function of a test-support package is a root (`pattern/PATTERN-test-only-code.md:26`).
The current tree passes; the gap is latent.

### [LOW] The guard's exemptions and import bound differ from the pattern text

**Location:** `pattern/PATTERN-test-only-code.md:24-37`

**Issue:** The guard skips generated files (`ast.IsGenerated`, line 163).
It also skips unexported empty marker methods (`isMarkerMethod`, lines 239-247).
The pattern's Enforcement section states neither.
The third bound says "a package no `main` imports is outside the analysis".
In fact the guard loads the test-support packages as initial patterns, so a production package only they import is analysed and counts as reached.

### [NIT] Test names keep the removed seam names

**Location:** `internal/configcli/configcli_test.go:191`

**Issue:** `TestRunCLIIn_FromNonGitDirectory`, `TestRunCLI_GroupGuard_NoGitRepoNeeded`, `TestRunCLI_GroupGuard_OutsideGitRepo` and the other `TestRunCLI_*` tests in the cli packages name functions that no longer exist.

### [NIT] TestNewPreflight_CarriesToldName names a test helper as its subject

**Location:** `internal/preflightshed/preflight_test.go:28-49`

**Issue:** `NewPreflight` now lives in the test file.
The test and its keep reason describe it as the subject, when the live code it pins is `NewPreflightWithChecks`.

### [NIT] The board benchmark doc names boardcli.RunCLI

**Location:** `docs/benchmarks/board-performance.md:93`

**Issue:** The line says the CLI rows "drive `boardcli.RunCLI`", which the diff deleted; `bench_cli_test.go` now calls `clihelp.Execute`.

### [NIT] The Prompts allowlist reason lists only some of its users

**Location:** `cmd/lym/testonlycode_linux_test.go:67`

**Issue:** The reason names the tests of `internal/darnengine` and `internal/landingshed`.
`seatengine.Prompts` is also used by the tests of `internal/burlerengine`, `internal/driftengine`, `internal/loomcli` and `internal/loomengine`.
A partial list of consumers goes stale; naming the group does not.

### [NIT] tmuxProcessName's comment reasons about a host the file never compiles on

**Location:** `internal/reedengine/proctree_windows.go:69-70`

**Issue:** The comment justifies splitting on both separators by hand, because `filepath.Base` on a host that is not Windows would not split `\`.
The function now sits in a Windows-only file, where `filepath.Base` handles both separators.

## Rejected

- code-tests F5 (`TestTmuxProcessName` now runs on Windows hosts only): `PATTERN-test-only-code`'s Windows-file bound requires this move, and nothing in the diff can restore Linux coverage without breaking that rule.
