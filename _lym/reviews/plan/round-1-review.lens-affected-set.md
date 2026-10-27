```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and which cards cover it.

**Per change: the affected set and the cards that cover it**

- **`HistoryEntry.Machine` (card 1).**
  - Set: `stepLocked`, `enter` and `rerouteStuck` appends, `TestStatus_JSONRoundTrip`, `shedengine/doc.go`.
  - `Goto` and `Relaunch` stay unstamped, which the card states.
  - No batten test was found that pins history entries with `DeepEqual`.
  - Covered by card 1.
- **`battenCLI.machineName` / `BuildShed` (card 2).**
  - Set: `specFor`'s `BuildShed`, and the hand receivers in `predrive_test.go`, `run_test.go`, `continue_pass_test.go` and others, whose nil seam now reads the real hostname.
  - Covered by card 2, as far as it goes.
- **`DobbyLandings` re-sign (card 3).**
  - Set: its one caller, `hubDobbyDeps` at `dobbydeps.go:154`, plus the `Deps.Landings` doc, `dobbyengine/doc.go`'s tidy bullet and `beamcli/doc.go:108`.
  - Covered by card 3.
- **`State.LandingsMachineOnly` (card 4).**
  - Set: `countLandings`, `TestCountLandingsAndTidy`, `doc.go`.
  - Covered by card 4.
- **`Watcher.CheckWith` (card 5).**
  - Set: `Check`'s callers are unchanged by the delegation, and `TestWatcher_Check`.
  - Covered by card 5.
- **`Deps.Log` signature, `SpawnDetached` output file, `Geometry` paths, `hubDobbyDeps` re-sign (card 6).**
  - Set: `passes_test.go:211` (the Log fake), `loop_test.go` harness (no Log), `dobbygeom_test.go:94`, `beamcli/doc.go:117`, and the one `hubDobbyDeps` caller at `cli.go:384`.
  - Covered by card 6; card 9 deletes the Log test.
- **`CheckBinary` signature, `Run` start path, registration order (card 7).**
  - Set: `loop_test.go:86` (the `CheckBinary` fake), `runDobby`, `dobbyengine/doc.go:24-26`, and `beamcli/doc.go:97-100`.
  - Covered by card 7, except `beamcli/doc.go`, which it lists under `Extends` (finding 3).
- **Worktree-gone exit, `PrimeTidyShuttle` (card 8).**
  - Set: `loop`, `recordStart`, `tidyShuttleFake`, `hubDobbyDeps`, both docs.
  - Covered by card 8.
- **Log events, `passKind.String` deletion (card 9).**
  - Set: the one `String` caller at `passes.go:251`, `docs/overview.md:448`, `dobbyengine/doc.go`, and the tick-failure and tidy tests.
  - Covered by card 9.
- **`placeDobby` re-sign, `runDobbyWith`, stdio reopen (card 10).**
  - Set: the one `runDobby` caller at `cli.go:365`, and `TestDobbyCmd_RefusesAPathItCannotPlace`, whose assertions are unchanged.
  - Covered by card 10.
- **`adopt` gate key and `seedAdoptGate` in `MigrateConfig` (card 11).**
  - Set:
    - `template.yaml`, `goReadGates` → `TiersValue`. The `twoTier` fixture at `config_test.go:621` holds no `adopt`, so that subtest is unaffected.
    - `TiersValue` reaches the loom and webster prompts (`loomcli/wiring.go:410`, `webstercli/wiring.go:259`).
    - `envkit.fixedGateLists`, `TestMigrateConfig`, `TestGateTest_Scopes`.
    - Every caller of `gateslot.MigrateConfig` through `configreg`'s `Migrate` hook:
      - the full-file reconcile at `configsync.go:507`;
      - the sparse hub base file at `configsync.go:417`;
      - the **local layer** file at `configsync.go:424` → 562;
      - **`sparseSeed`** at `configsync.go:692`;
      - **`PrepareRunSettings`** at `runsettings.go:73-79`.
    - `pattern/PATTERN-test-speed.md`'s Gate keys table.
  - Card 11 covers the hub base file only (finding 1 and finding 4).
- **`RunGate` / `TierObserver` (card 12).**
  - Set: the `report`, `reportBlocked` and `runWithSlots` call sites; CLI calls pass the zero observer.
  - Covered by card 12.
- **Hidden `adopt check` and `Args: ArbitraryArgs` (card 13).**
  - Set: the `cmd/lym/clitree_test.go:216` hidden-verb audience check, which the card's internal audience satisfies.
  - Its files live in `AdoptScratchDir`, whose doc comment (`fabricengine/junctionnames.go:164`) names only the suite log.
  - Covered by card 13, except that doc (finding 4).
- **`startSuite` re-sign, `suiteStart.Record`, warning text (card 14).**
  - Set: `TestStartSuiteStartsNothingUnderTest`; `TestAdopt` at `adopt_test.go:203,609,676`, where `testSuite` gets no `Record` from the fake, so it stays equal; `cli.go:44` step 8; `adopt.go:120,327`; `doc.go:73-77`; `docs/overview.md:438`.
  - Covered by card 14.
- **`adopt_check` in `lym beam status` (card 15).**
  - Set: `statusResult`, `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`, and `statusCmd`'s Long text, including its "its one write" claim at `cli.go:72-73`.
  - Covered by card 15, except the write claim (finding 2).

**Findings**

1. **MEDIUM, class scope.** Card 11 (`internal/gateslot#MigrateConfig` / `seedAdoptGate`).
   The seed runs everywhere `MigrateConfig` runs, but the card covers only the hub base file.
   - Members it missed:
     - the local layer file (`configsync.go:424` → `reconcileSparseFile` at 562-563), which gains an `adopt` Decision 9 does not license;
     - `sparseSeed` (`configsync.go:690-694`);
     - `configreg.PrepareRunSettings` (`runsettings.go:73-79`).
   - In `PrepareRunSettings`, a profile or run_settings `gate` object that holds `gates.finalize` and no `adopt` logs `gates.adopt: added with the value of gates.finalize; way forward: rename or remove the key…` at Warn.
     Per `runsettings.go:43` that Warn repeats at every read, and its way forward is wrong for an addition.
   - No test row covers any of these callers.
   - Evidence: `internal/configsync/configsync.go:417,424,507,562,692`; `internal/configreg/runsettings.go:43,73-79`.
2. **LOW, class consistency.** Card 15 (`internal/beamcli#statusCmd`).
   `ReadCheckView` → `ReadCheck` → `state.ReadJSON` runs `MkdirAll` on the record's directory and takes its read lock (`state.go:96-101`).
   So every `lym beam status` creates `_board/.lym/adopt/` and a lock file.
   That falsifies `statusCmd`'s Long text, "It starts and changes nothing; its one write is the empty orch directory its watcher probe creates" (`internal/beamcli/cli.go:72-73`).
   Card 15 edits that text only to name the new section.
3. **LOW, class consistency.** Card 7 (`internal/beamcli/doc.go` under `**Extends:**`).
   The card moves `registerDobby` after the lock and drops the `Live` pre-check.
   That rewrites the existing sentences at `internal/beamcli/doc.go:97-98`, which say it registers before `Run`, and line 100, "before anything is registered or locked".
   A rewrite is not what `Extends` covers, so the ref belongs under `**Edit:**`.
4. **NIT, class consistency.** Cards 11 and 13 miss two docs in the set:
   - `pattern/PATTERN-test-speed.md`'s Gate keys table lists every key with its site, and no card adds `adopt`;
   - `fabricengine.AdoptScratchDir`'s doc (`internal/fabricengine/junctionnames.go:164`) says the directory "holds the log of the test suite", and no card mentions `check.json`, `check.out` or `check.lock`.

**Tool calls**
1. Grepped the dobbyengine and beamcli tests for `Log` and `CheckBinary` fakes, the callers of the re-signed `hubDobbyDeps`, `placeDobby` and `runDobby`, and the `twoTier` fixture: the manifest held their definitions but not every fake and caller.
2. Grepped tests and fixtures naming `finalize`, the adopt suite envelope asserts and CLI-tree tests: needed to find readers of the gate keys, the `suite` envelope and the verb tree.
3. Grepped the hubforge, configsync and hubreconcile fixtures for a tuned `finalize`, and `clitree_test.go` for the hidden-verb rule: needed to see whether the seed changes fixture hubs and whether the hidden verb passes the audience check.
4. Grepped tests and docs naming `dobby.log` or `LogPath`: needed to find readers of the moved output and the dropped pass lines.
5. Grepped batten tests that build a shed for exact-history asserts: needed to see whether card 2's real-hostname stamp breaks a test.
