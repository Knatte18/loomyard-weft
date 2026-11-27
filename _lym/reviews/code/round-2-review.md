---
verdict: BLOCKING
findings:
  - id: F12
    severity: BLOCKING
    class: scope
    location: pattern/PATTERN-test-speed.md:152
    summary: "The changed PATTERN-test-speed hunk says a test that builds a real .NET project names upkeep, and line 157 says a watched guard stays at the task gates. TestCSharpFixtures_CompileWithDotnet (cmd/lym/csharpfixtures_integration_test.go:54) fits both. It is a watched cmd/lym guard that walks the whole repository from scankit.Root and runs dotnet build per fixture directory, and it carries no //lym:only. So it still runs in full at every scope-all task gate (webster, tiers 0-2 on this hub), where watch skips do not apply, and on any *.cs change at a narrowed one. The tree breaks the declared rule, and the record's ask to name a place for every whole-repo guard of cmd/lym is left undone for this guard. The change description's claim that a watched guard reads a bounded set of files and runs only when one of them changes is false for this guard, and false at every whole-scope gate."
    fix: "Add the directive line `//lym:only upkeep` to cmd/lym/csharpfixtures_integration_test.go, directly above `func TestCSharpFixtures_CompileWithDotnet`, stacked with its `//lym:watch *.cs` and `//lym:guard` lines. Drop the doc comment's sentence `A narrowed gate runs it only when a C# file changes; a whole gate always runs it.` and write `Only the upkeep gate key runs it.` in its place. In pattern/PATTERN-test-speed.md:152, name the three tests instead of the general phrase: the dead-code guard of PATTERN-test-only-code, `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` in internal/gatecli, and the C# fixture compile `TestCSharpFixtures_CompileWithDotnet` in cmd/lym. In pattern/PATTERN-test-speed.md:157, write `- A watched guard that reads a bounded set of files, and a guard of any package but cmd/lym, stays at the task gates.` In _lym/landing/summary.md, add `TestCSharpFixtures_CompileWithDotnet` to the upkeep bullet under Where each check now runs. In the same file, change the design-decision bullet on watched guards so it says a watched guard reading a bounded set of files stays, and that a scope-all gate runs it in full."
prior:
  - id: F1
    status: resolved
    answer: "markedScope now refuses a marked test of a tier no listed tier is or covers, naming the tests and gate.gates.upkeep, and refuses a call that would run none, for a missing go.mod and for no marked test alike. TestGateTest_Scopes asserts each refusal. Both refusals sit before planning, which settles the finding as well as the runTest site it named. The upkeep stencil says what to file for each refusal."
  - id: F2
    status: resolved
    answer: internal/gatecli/doc.go and internal/gateslot/template.yaml now write gate.gates and gate.gates.upkeep in full.
  - id: F3
    status: resolved
    answer: scopeInvocations opens with the scopeMarked case returning no invocation, and TestPlanTierRuns holds the --no-carry row.
  - id: F4
    status: resolved
    answer: The UpkeepCall closure returns no call under a scoper other than go, and the beamcli wiring test asserts both the none and go cases.
  - id: F5
    status: resolved
    answer: TestRemovedVerbs_NamedNowhere and TestAgentMessaging_NoEngineTool now carry //lym:guard and //lym:only parent-align finalize.
  - id: F6
    status: resolved
    answer: The change description now calls the door one-way and names both effects and their recoveries.
  - id: F7
    status: resolved
    answer: "A card-scope row puts a held //lym:guard test outside the scope and asserts the guard step leaves it out. The wantHeld subtest reads the Held of the entries a real run and an inert carry write. The tidy test covers an unset, a failing and a rendering UpkeepCall, and the beamcli test covers prime and beam wiring."
  - id: F8
    status: resolved
    answer: Rows now cover skippedInside with a covering tier, greenRecord.with over entries differing in Held, the none-scoper refusal, explicit targets beside the marked key, the -C scan directory and an unlistable module.
  - id: F9
    status: resolved
    answer: withOnlyTests leaves request.held empty when the words after -- carry a filter flag, so the listed runs and the guard step agree.
  - id: F10
    status: resolved
    answer: withOnlyTests takes preCtx into ModuleOnlyTests and modulePackageDirs, which run go list under exec.CommandContext, and runTest checks preTierExpiry right after it.
  - id: F11
    status: resolved
    answer: The dobbyengine doc bullet now says before its report, and the board_tidy_timeout_min line sits back under the spawn bullet.
---

### [BLOCKING] The C# fixture compile guard has no place, against the declared PATTERN text and the record

**Location:** `pattern/PATTERN-test-speed.md:152`, with the broken site at `cmd/lym/csharpfixtures_integration_test.go:54`.

**Issue:** The amended board entry asks that every whole-repo guard of `cmd/lym` be named a place, landing or upkeep.
The declared background text sends a test that builds a real .NET project to `upkeep`,
but `TestCSharpFixtures_CompileWithDotnet` stays unmarked.
That test walks the whole repository from `scankit.Root` (its walk skips only `.git`, `_lym`, `.lym` and similar directories),
and it runs `dotnet build` on one throwaway project per fixture directory.
Its `//lym:watch *.cs` saves nothing at a scope-all gate.
Watch skips apply only to a narrowed share and a `--card` scope,
so the webster key, a task gate that runs tiers 0 to 2 here, runs this guard in full every time.
Line 157's watched-guard exemption contradicts line 152 for this test,
and the change description's reason for the exemption, that a watched guard reads a bounded set of files, does not hold for it.
My sweep of `cmd/lym` found no other whole-repository walker that carries neither a place nor a watch marker.
The other watched guards scan fixed package roots or named files.

## Focus verdicts

- Landing routes: not a defect.
  Every recipe that lands code ends in a `Finalize` row whose gate key is `finalize`, its row name in lower case, with no `gate_key` override: tapestry, darn, sampler, hardener and pawl (the beam's own landing).
  `Finalize.landUnderLock` runs `mergeAndGate` before the parent-side merge and before any GitHub merge, re-entry over an already-landed parent included.
  verifytree skips a run only for the same command over the same tree, so a webster pass cannot stand in for `--gate finalize`.
  The green record carries Parent-Align's pass to Finalize only when the held sets match, both being the upkeep tests.
  The batten `Land` row only pulls the prime after the inner run's Finalize, and the arras, raddle and motif recipes land no code.
  So the tests marked `parent-align finalize` run on the merged tree that lands, under this hub's `finalize: {tiers: [0, 1, 2]}`.
- Marked-scope key: not a defect.
  `markedScope` refuses a non-go scoper, a marked test of an unlisted tier, a missing `go.mod` and a module with no test marked for the key, each with a non-zero exit.
  `scopeInvocations` plans no listed invocation under `--no-carry` or `carry: false`.
  A filter flag after `--` empties `held`; the guard step's own `-run` comes last, so the marked tests still run.
  Explicit targets replace the scope by the documented contract.
  A hub that declares no tier returns before the scan, running only lints.
  That is the documented behaviour, and no test can run there.
- Held tests and the green record: not a defect.
  `covered` and `newestWholeAncestor` require the entry's `Held` to be a subset of the request's, and `with` keys entries on `Held`.
  A guard-step entry carries its `-run` filter as args, so it matches only an identical filter.
  The card-scope row with `TestLandingGuard` fails if `withoutHeld` stops dropping it, since the guard invocation would gain `./heldpkg`.
  The `wantHeld` subtest fails if `recordPass` or `recordInertCarried` stops writing `Held`.
- Board-tidy upkeep call: not a defect.
  The call renders only on the prime's dobby and only under the go scoper, and a render error falls back to the no-upkeep step.
  The upkeep fragment says what to file for a failing test, a dead function, a refusal for no marked test and any other refusal.
- Whole-repository scans: one defect, F12. No unmarked, unwatched whole-repository walker remains in `cmd/lym`.
  The unmarked tests left there scan fixture trees, read single files or test the CLI tree.

## Rejected

- The board-tidy agent may hit `board_tidy_timeout_min` (30) while the upkeep gate waits for slots and builds the module.
  Dobby then posts a no-report line to the orch, so the loss is visible.
  This is not a BLOCKING defect, and it sits on no hunk this round changed.
- On this hub the upkeep call stays refused until `gate.gates.upkeep` lists tier 1.
  The change description states this, and the refusal reaches the board as a bug note through the stencil.
  This is the outcome F1's fix asked for.
