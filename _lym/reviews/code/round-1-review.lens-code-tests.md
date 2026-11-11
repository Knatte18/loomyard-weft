```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour**
- `consoletitle.Title` shape `<lead> [<shortname>:<project>] <name>` is pinned by `TestTitle` (`consoletitle_test.go`) rows `EveryPart`, `AnOrchLead` and `AnUpperCaseShortnameIsLowered`.
- `Title` leaving out a missing part and its separator is pinned by `TestTitle` rows `NoShortname` (`T7 [P3] slug`), `NoLeadNorProject` (`[ly] stray`) and `TheNameAlone`.
- `Title` with a lead but neither shortname nor project (`T4 slug`) has no row. `ConsoleWindowTitle` can't produce that input, because a lead is set only for a board entry and a board entry always gets at least `P0`. I file nothing for it.
- `consoletitle.Workspace` bracket form `[ly] loomyard workspace` is pinned by `TestWorkspace` rows `AShortnameOpensTheTitle` and `NoShortname`.
- The workspace title bracket form in the generated workspace files is pinned by `TestRegenerateHubWorkspace` (`fabricengine/hubworkspace_integration_test.go:363-364`) and by the ideengine spawn integration checks (`spawn_hub_integration_test.go:338` and `:534`).
- `ConsoleWindowTitle` for the prime (`ORCH [ly:P0] root`, naming the root project rather than the worktree) is pinned by `TestWindowTitles` rows `ThePrimeIsTheOrchsOfTheRootProject` and `NoBoardStillTitlesThePrime`.
- `ConsoleWindowTitle` for a beam (`ORCH [ly:P2] bm`) is pinned by `TestWindowTitles` row `ABeamIsTheOrchsOfItsProject`.
- `ConsoleWindowTitle` for a task is pinned by these `TestWindowTitles` rows:
  - in a project: `ATaskInAProject`;
  - under root, with `P0` as the fallback: `ATaskUnderRootNamesTheRootProject`;
  - under a beam: `ATaskUnderABeam` and `ATaskInAProjectUnderABeamTakesTheBeam`;
  - under a done beam: `ATaskUnderADoneBeamTakesTheBeamsHandle`.
- A task whose project is missing from the board (`T10 [ly] orphan`, with no project handle) is pinned by `TestWindowTitles` row `ATaskWhoseProjectIsMissingLeavesOutTheProject`.
- A worktree off the board getting no lead (`[ly] stray`) is pinned by `TestWindowTitles` row `AWorktreeOffTheBoard`. This is the new `onBoard` branch.
- With no hub shortname, `TestWindowTitles` row `NoShortname` pins `T7 [P3] gate-shard`.
- With no prime name, `TestWindowTitles` row `NoPrimeNameTitlesThePrimesSlugOffTheBoard` pins `[ly] main`.
- An unreadable board is pinned by `TestWindowTitles` rows `AnUnreadableBoardLeavesOutTheHandles` and `AnUnreadableBoardTitlesABeamBySlug`.
- The merge of `IDEWindowTitle` and `TerminalWindowTitle` is pinned by the single `ConsoleWindowTitle` assertion in `TestWindowTitles`. At the terminal-window caller, `TestTerminalWindowTitle` (`reedcli/terminalwindowtitle_integration_test.go:29`) pins the prime row `ORCH [tst:P0] root`.
- `RecipeWindowTitle` is pinned by `TestRecipeWindowTitle` rows `ATaskInAProject`, `ABeamTakesNoLead`, `AWorktreeOffTheBoard` and `AnUnreadableBoardLeavesOutTheHandles`.
- The task console title through `{title}` (`battencli` `commandDeps`) is pinned by `TestCommandDeps_TitleIsTheTasksConsoleTitle`, including the `T4 [P0] run` and `T4 [ly:P3] run` rows.
- The run status window title (`loomcli`) is pinned by `TestStatusWindowTitle` and `TestStatusStrandSpec_TitlesTheWindowWithTheRunsHandleAndRecipe`.
- The VS Code look title at the `ideengine.lookFor` caller is pinned by `TestLookFor` rows for the prime, both beams, `t1`, `t3` and `t5`.
- The settings title the VS Code spawn writes is pinned by `spawn_clean_integration_test.go:139` (`[tst] <slug>`) and `spawn_hub_integration_test.go:561` (`ORCH [tst:P0] root`).
- The slug-after-bracket titles at the `ideengine.lookFor` caller, and the doc and template text, need no test: the ideengine rows above cover the former, and prose is exempt.

**Findings**
1. NIT, class `consistency`: `internal/loomcli/smoke_drivingsurface_test.go:254` builds `wantName` with `consoletitle.Title("", hubforge.TestShortname, "", shedrun.RecipeTapestry)`. That is the production function, so the assertion cannot fail on a format regression. `TestRecipeWindowTitle` already pins the shape. A literal `[tst] tapestry` would pin the window name without that circularity. Optional.

No uncovered behaviour and no duplicated test found. `TestTitle` and `TestWindowTitles` rows overlap only where the layers differ: the builder versus the board lookup. The `//testtiming` redundancy report lists none of these tests as contained.

**Tool calls:** none. The inherited context already held the diff and the test files.
