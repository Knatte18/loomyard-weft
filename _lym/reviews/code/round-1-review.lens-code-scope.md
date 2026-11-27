```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each change in the diff to a goal point of the board entry and each goal point to the changes that build it.

**Goal points** (the record holds no `Added after Discussion` entries)

- G1: whole-repo guards that catch a break, among them tree-sitter linking and the cross-compile, leave the task gates and run once at landing.
- G2: an upkeep run goes with the board-tidy agent dobby spawns every `board_tidy_every` landings, on the main branch.
  - G2a: the dead-code (call-graph) test runs there, and each dead function becomes a board note.
  - G2b: the .NET tests `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` run there, and a failure becomes a bug note.
- G3: find every whole-repo guard in `cmd/lym` and name where each runs, at landing or in the upkeep run.
- G4 (changed since `165f91c6b`): decide whether the `create-caller-later` plan check stays.
- G5 (changed since `a69580709`): moving guards off the task gates must also cover the widened card scope.
- G6 (folded B2): a computed or `--base` scope must stop skipping the guard tests of an impacted package; place every package's guards.
- The memory-guard section and the coverage-map tools left to `remove-testmap` are context only and ask for no change.

**Per change**

- C1, the `impactset` marker scan (`only.go`, `FindOnlyTests`, `ModuleOnlyTests`, the `modulePackageDirs` extraction, `doc.go`, tests): the mechanism for G1, G2 and G3.
- C2, `gateslot` (`ScopeMarked`, `UpkeepGateKey`, scope decoding, template key and comments, `doc.go`, config tests): G2.
- C3, `gatecli` held and marked tests (`withOnlyTests`, `markedScope`, `withoutHeld`, the held skip in `planTierRuns`, `isMarkedCall`, help text, `doc.go`, the `tiers_test` and `scopes_integration_test` rows): G1, G2 and G5. Every `--gate` call holds the tests back, the card and batch keys included.
- C4, the `held` field of the green record (`green.go`, `greenArgs`, `newestWholeAncestor`, `diffBaseGroups`, `green_test`): forced by C3, so a pass that held tests back never carries over them.
- C5, `Targets.InsideGuards` and `skippedInside` in `computeScope` (with the `doc.go` "Bound" removed, `targets_test` and the scopes row): G6.
- C6, `//lym:only parent-align finalize` on the unwatched `cmd/lym` guards, `TestGofmt` included, plus tree-sitter and the cross-compile: G1 and G3.
- C7, `//lym:only upkeep` on `TestTestOnlyCode_…`: G2a. The same marker on the two `gatecli` .NET tests: G2b.
- C8, dobby `UpkeepCall` (`loop.go`, `tidy.go`, `doc.go`, `tidy_test`), the prime-only wiring in `beamcli/dobbydeps.go`, the two new beam fragments, the template's step 3 and 4, and their `stencils.go` registration: G2, G2a and G2b.
- C9, retiring the plan check (`latercaller.go` and its test deleted, the `validate.go` call removed, both specs, both loom stencils, and `cmd/lym/testsupportdirs_linux_test.go` deleted, since its list only served the check): G4.
- C10, `PATTERN.md` and the `PATTERN-test-speed` and `PATTERN-test-only-code` backgrounds: forced by C3, C6 and C7.
- C11, commit `011874f` (comment breaks, help length, gofmt): forced by the changes above.
- No change traces to nothing, and no `Pattern-Departure:` trailer is needed.

**Per goal point**

- G1: C1, C3, C6.
- G2: C2, C3, C8.
- G2a: C7 and C8, but not effective on this hub (finding 2).
- G2b: C7 and C8, with the same limit (finding 2).
- G3: C6 and C10, partial (finding 1).
- G4: C9.
- G5: C3.
- G6: C5.

**Findings**

1. MEDIUM, scope. Location: `cmd/lym/removedverbs_test.go:40`, `cmd/lym/agentmessaging_test.go:95`.
   - G3 is only partly built: two whole-repository scans in `cmd/lym` carry no `//lym:guard` marker, so the diff gives them no place, and they still run in every task gate that impacts `cmd/lym`.
   - Evidence: `TestRemovedVerbs_NamedNowhere` calls `scankit.Walk(t, scankit.Options{}, scan)` over the whole repository and adds the stencil and skill roots (`removedverbs_test.go:77-79`). `TestAgentMessaging_NoEngineTool` walks the stencils, the prompt fixtures and every Go file (`agentmessaging_test.go:167,181,192`).
   - G3 asks for every whole-repo guard in `cmd/lym`, and the diff only covered the ones carrying the marker.
   - Fix: name each test's place, most likely `//lym:only parent-align finalize`, or say why it stays at the task gates.
2. MEDIUM, scope. Location: `internal/gateslot/template.yaml:41`.
   - G2a and G2b do not take effect on this repository as landed: the template's `upkeep` key lists `tiers: [0]`, and all three upkeep tests are tier 1.
   - Evidence: `TestTestOnlyCode_…` calls `tiergate.Require(t, 1)` (`testonlycode_linux_test.go:253`); so do `TestGateTest_DotnetFixture` (`dotnetfixture_integration_test.go:100`) and `TestGateRedFirst_Dotnet` (`redfirstdotnet_integration_test.go:443`). `markedScope` names a test of an unlisted tier on stderr and skips it (`only.go:387-390`).
   - So the board-tidy upkeep call runs none of them until someone sets `gates.upkeep` to `{tiers: [0, 1], scope: marked}`. The change description admits this gap, but nothing in the branch closes it.
   - Fix: make that setting part of landing this task (it is a hub config change, outside the diff), or record a ruling that defers it.

Tool calls:
1. Grep of the `cmd/lym` test files that scan the repository but carry no guard or only marker: the reading manifest had covered only the marked guards, not unmarked scans.
2. Grep of the test functions and `scankit` calls in the four unmarked files from call 1, to tell real whole-repo scans from reads of `main.go` alone.
