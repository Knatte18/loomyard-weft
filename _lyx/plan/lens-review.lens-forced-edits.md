```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits — sites each card's change makes wrong that no card lists.

**BLOCKING — card 16 (shedrecipe registry)**: `minimalEntries` needs rows for the three new entries.
`TestEveryEntry_RejectsUnknownConfigKey` loops over `Names()` and fails any entry with no row in `minimalEntries` (`internal/shedrecipe/conformance_test.go:100-108`).
Card 16 adds `BeamUp`, `BeamAwait` and `BeamDown` to the registry but lists neither that test nor `minimalEntries`.

**BLOCKING — card 16 (shedrecipe registry)**: `TestNames` pins the registered name set and is not listed.
It enumerates the names, e.g. `"BeamDescribe"` and `"DriftAwait"` (`internal/shedrecipe/registry_test.go:49-125`, lines 87 and 95).
The three new names fail it.

**BLOCKING — card 16 (loomrecipe)**: `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` is not listed.
It expects loom's own engines plus only `DarnWrite` and `BeamDescribe` (`internal/loomrecipe/recipe_test.go:216-245`).
The pawl recipe adds `BeamUp`, `BeamAwait` and `BeamDown` to `RecipeEngines()`, so the lengths differ and the test fails.

**BLOCKING — card 10 (orch messages)**: `TestStatus_RestartsADeadWatcher` asserts the old text and is not listed.
Its row `AFailingSpawnIsTheErrorEnvelopeNamingStart` asserts `wantError: "lyx orch start"` (`internal/orchcli/cli_test.go:664`).
Card 10 rewords `statusCmd`'s restart failure to name `lyx beam start`.

**BLOCKING — card 10 (orch messages)**: `TestWatcherIndexSource` asserts the old text and is not listed.
It asserts the error names `lyx orch start` (`internal/orchcli/watch_test.go:83-84`).
Card 10 rewords `indexWayForward`.

**BLOCKING — card 10 (verb removal)**: `TestOrchIntegration_Refusals` runs the removed verb and is not listed.
It runs `"start"` in its verb loop and in `RunCLIIn(anchor, &out, []string{"start"})` (`internal/orchcli/refusal_integration_test.go:36` and `:101`).
`start` no longer exists after card 10.

**MEDIUM — card 10 (verb removal)**: `TestSmokeOrch_OneFullCycle` runs the removed verb and is not listed.
It runs `orch start --no-attach` (`internal/orchcli/smoke_cycle_test.go:206`).
Card 10 lists only `liveFixture.startOrch`.

**MEDIUM — card 17 (`writeBoardEntrySnapshot`)**: `TestWriteBoardEntrySnapshot` is not listed.
It is at `internal/loomcli/wiring_darn_test.go:353-391`, and card 17 makes `writeBoardEntrySnapshot` serve the pawl recipe too.
No row covers the pawl case.

**LOW — cards 1 and 21 (way forward between cards)**: `createSteps` names a removed verb from card 10 until card 21.
Card 1 keeps the `lyx orch start` way-forward wording, which `TestCreate` asserts (`internal/beamcli/create_test.go:291`).
Card 10 removes the verb, so from card 10 to card 21 create's way forward names a command that no longer exists.
Card 10's sweep of "every message naming `lyx orch start`" leaves this one out.

**LOW — consistency (overview and cards)**: many cross-card references point at the wrong card number.
- Card 1 says "card 19 rewrites create"; it is card 21.
- Card 3 says `RecipePawl` is declared by "card 5" and moved by "card 23"; they are cards 6 and 25.
- Card 4 says "cards 6 and 22"; they are cards 7 and 24.
- Card 7 says "cards 8 and 9 replace"; they are cards 10 and 11.
- Card 7's `--all` is attributed to "card 9"; it is card 11.
- Card 10 says adopt is "card 22"; it is card 24.
- Card 17 says `lyx beam land` is "card 20"; it is card 22.
- Card 18 says `StartLanding` is removed by "card 20"; it is card 22.
- Card 5 says the recipe test that pins `RunHolds` is in "card 16"; that is correct.
- The overview's Shared Decisions put `TestBoardRecipe` in card 6; that is correct.

**LOW — card 13 (`BeamLandBlockers`)**: the exported struct's `Members` field has the unexported element type `beamMember`.
Card 17's `pawlLandBlockers` (loomcli) and card 23's `beamLandReading` (beamcli) build values from it in other packages.
Exporting the element type, or naming it in the card, would make that clearer.

Further tool calls:
1. Grepped for callers of every removed or re-signed symbol: card coverage had to be checked across the tree, beyond the files pre-read.
2. Grepped orch tests for `"start"` and `orch start`, the removed seams, and the shedrecipe test tables: test bodies were not in context.
3. Ran `lyx quarry` over shedrecipe, loomcli and pairteardown tests and read `cli_test.go:660-668` and `beamland_integration_test.go:100-128`: to name the test functions that hold the forced sites.
4. Grepped `OwnerOf` fakes, the conformance loop and the remaining `lyx orch start` assertions: these call sites were not in context.
5. Read `recipe_test.go:216-245`, `conformance_test.go` and the coverage guard: to confirm whether the engine-set and registry tests fail.
