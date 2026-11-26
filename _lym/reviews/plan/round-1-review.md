---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-reconcile-before-load.md:7
    summary: Card 9 runs the walk only on the lym loom pre-run and three direct loomengine.LoadConfig callers, so lym shed (loom recipes via loomcli.ArmAt), batten verbs other than run (configreg.SettingLookup of tapestry) and lym fabric merge verbs (websterInFlightWarnings) still load the loom or tapestry config strictly with no walk, breaking D5.
    fix: "In card 9, add to **Edit:** `internal/shedcli#shedCLI.resolvePersistentPreRun`, `internal/battencli#battenCLI.arm` and `internal/fabriccli#websterInFlightWarnings`, and add `internal/shedcli/doc.go` and `internal/fabriccli/doc.go` under **Uses:**. In the Intent, add: shed's pre-run calls `hubgeom.ReconcileBeforeLoad` before `armFromSeed` for every shed verb; `battenCLI.arm` calls it for every verb in place of its `verb == \"run\"` call; `websterInFlightWarnings` calls it before its `SettingLookup`. Restate the method: the sites are every route to a strict loom or tapestry load, meaning `loomengine.LoadConfig` or `LoadTapestryConfig`, `loomcli.ArmAt` and `configreg.SettingLookup` or `configengine.Load` on the loom or tapestry spec, not the direct `LoadConfig` callers alone. Add a row to `TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads`, or to a shedcli tier1 test, in which `lym shed step` on a loom recipe over a stale stamp resolves `loom.code.max_bounces` to 2. validate-plan --overlay with these targets reports no new blocking finding."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-board-run-settings-carry.md:35
    summary: Card 8 claims boardCriticalSection commits and pushes and restores the saved store when the hook after the save fails, but boardCriticalSection returns on an afterSave error with board.json already saved and no restore, and it commits only through a detached spawnSync, so D7's 'a failed board write leaves the entry as it was' and the EnsureFailingRunSettingsCarry row cannot hold as planned.
    fix: "Rewrite card 8's RewriteRunSettings paragraph from the code as it is (internal/boardengine/board.go:181-199). RewriteRunSettings snapshots board.json and the rendered board files before its boardCriticalSection call and writes them back itself when beforeCommit, which is boardCriticalSection's afterSave, or any later step returns an error, so a failed write leaves the board's bytes as they were. Replace 'saves, commits and pushes as the board's write path does' with: it saves and renders, and the board's detached sync commits and pushes. State that a commit or push failure in that sync never reaches the walk and is retried by the board's own sync, so 'a failed read or write' in D7 means one up to the save and render. Keep the EnsureFailingRunSettingsCarry row asserting the entry is unchanged, which the restore now makes true."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/09-reconcile-before-load.md:31
    summary: Card 9 puts the walk before the pre-run's skip of the checklist and pusher verbs and bare lym loom. D5 licenses it only for verbs that load the loom or tapestry config, and the pusher, which every run commit starts and which runs from the hub's Bolt directory (internal/loomcli/pusher.go:24-26), would take the hub lock and walk every worktree on a stale stamp, a bound the card never states.
    fix: "In card 9, call `hubgeom.ReconcileBeforeLoad` in `loomCLI.resolvePersistentPreRun` after the skip for `loom`, `checklist` and `pusher` and before the `inReviewGroup` skip, so the circling, decision, parent-review and peers groups still walk but the pusher, the checklist and the bare group do not. Rewrite the bullet at line 31 to say so, and state in the Bound line that the pusher and checklist verbs run no walk because they load neither config."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: internal/hubreconcile/doc.go:3
    summary: After card 9, hubreconcile/doc.go still says a start verb is what calls Ensure through EnsureOrWarn (line 3) and that the next start verb retries a skipped walk (line 33), and card 8 adds a new skip cause, the run_settings carry. Card 9 lists the file only under Uses, and cards 6 and 8 only extend it.
    fix: "In card 9, move `internal/hubreconcile/doc.go` from **Uses:** to **Edit:**. In the Intent, rewrite doc.go line 3 to say every verb that loads a reconciled config calls Ensure through EnsureOrWarn, and line 33 to say the next such verb retries, naming the run_settings carry among the skip causes. This move was validated together with F1's targets."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/02-config-moved-keys.md:61
    summary: pattern/PATTERN-config-layers.md line 7 says no step removes an unknown base-layer key except the operator's lym config reconcile or --unset. Card 6 makes every hub walk rename and drop the moved old keys in hub base files, yet card 2 edits only the opening line, the 'Keys the template lacks' section and the reinterpretation rule, and card 14 changes key names only.
    fix: "In card 2's paragraph on `pattern/PATTERN-config-layers.md`, add that the rule on line 7 gains an exception: a key the module's move table names is moved, or dropped when null or when its new path is set, by every reconcile, the hub walk included. Also amend the sentence at lines 77-78 that says every other reconcile keeps such a key."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/04-nest-loom-tapestry.md:121
    summary: Card 4 says TestModules_EveryTemplateKeyClassified and TestModules_LocalLayerRefusals 'run over the nested paths unchanged', but the first test's wantLocalDenied table lists flat loom and tapestry keys such as code_auto_accept and plan_lenses (internal/configreg/configreg_test.go:175-176), which must move to the nested paths.
    fix: "In card 4, replace line 121 with: `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` loom and tapestry rows move to the nested paths, such as `code.auto_accept` and `plan.lenses`, and `TestModules_LocalLayerRefusals` runs over the nested paths."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/06-reconcile-moves.md:63
    summary: Card 6 says TestReconcile_FailureNamesItsFile's unparseable-file rows now get their error from the move's *FileError, but those rows use modules with no move table, which moveOldKeys returns unchanged. So moveOldKeys' own parse-failure branch has no reaching row.
    fix: "In card 6, replace line 63 with: `TestReconcile_FailureNamesItsFile` (tier0) gains a row whose unparseable `loom.yaml` fails in `moveOldKeys` with a `*FileError` naming that file; its existing rows are unchanged."
  - id: F8
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-moved-key-tables.md:50
    summary: Card 5's TestSeedChild_RunSettings row feeds Seed-Child a hand-written moved-key text through a fake PrepareRunSettings, so D4's acceptance criterion that Seed-Child prints exactly one way forward is never checked against the text refuseMovedKeys really produces.
    fix: "In card 5, make the `TestSeedChild_RunSettings` row's fake return the error that `configreg.PrepareRunSettings(map[string]any{\"loom\": map[string]any{\"code_max_bounces\": 2}})` returns, rather than authored text. The row then asserts the Stuck reason holds both full names and exactly one `way forward:`."
  - id: F9
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-board-run-settings-carry.md:49
    summary: carryHub's fixture writes board.json with no board config (internal/hubreconcile/carry_test.go:19-34). RewriteRunSettings opens the board from its board config, so the planned 'board that does not parse' row reaches skipped through the missing config, never through the unparseable board.
    fix: "In card 8, make the `TestCarryMovedRunSettings_UnreadableBoardIsSkipped` row write a valid board config beside the unparseable `board.json`, so the skip comes from the board read; or rename the row to say it pins a missing board config."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/07-profiles-moves.md:32
    summary: "Card 7's write rule contradicts itself: 'a move in the file alone does not count as a change on its own when the follow then writes the template's bytes' against 'the write happens when the follow or the move changed the file'. An untouched flat file whose follow reports no unit change must still be written, or it stays flat on disk."
    fix: "In card 7, replace line 32 with: the file is written when its bytes after the move and the follow differ from the bytes read, so an untouched flat file is rewritten nested even when the follow reports no unit change."
---

### [BLOCKING] The walk before a loom or tapestry load misses lym shed, batten and fabric

**Location:** `_lym/plan/09-reconcile-before-load.md:7`
**Issue:** D5 asks that every lym verb outside `lym config` that loads the loom or tapestry config run `hubreconcile.EnsureOrWarn` first.
Card 9 hooks the `lym loom` pre-run, `hubLandDeps`, `wireHub` and `loomConfigOf`, the direct `loomengine.LoadConfig` callers.
Three other routes load those configs strictly:
- `lym shed run|step|status|pause|goto` for every loom-contract recipe arms through `loomcli.ArmAt` (`internal/shedcli/table.go:51-94`, `internal/loomcli/arm.go:507-510`), which reaches `wire` and `loomengine.LoadConfig` (`internal/loomcli/wiring.go:598`) without passing the loom pre-run.
  The shed driver stencil drives runs through `lym shed`.
- Batten verbs other than `run` reach `configreg.SettingLookup` and so `configengine.Load` on the tapestry spec (`internal/battencli/wire.go:807`, `:1691`, `internal/configreg/settinglookup.go:55`).
  Only `verb == "run"` reconciles (`internal/battencli/arm.go:301`).
- `lym fabric` merge verbs read `tapestry.implementer` through `websterInFlightWarnings` (`internal/fabriccli/merge_verbs.go:36-45`).

On a new build with a stale stamp, each of these hits card 2's refusal of an old name with no walk having moved it.
The enumeration followed direct `LoadConfig` callers only, so the fix restates the method as well as adding the sites.

### [BLOCKING] Card 8 rests on a board write path that neither restores nor commits

**Location:** `_lym/plan/08-board-run-settings-carry.md:35`
**Issue:** `boardCriticalSection` (`internal/boardengine/board.go:85-201`) saves the store and then runs `afterSave`.
An error from `afterSave` returns `nil, err` with `board.json` already rewritten and no restore (`board.go:181-189`).
The function never commits or pushes itself: it renders and calls `spawnSync`, a detached sync (`board.go:191-199`).
So "the board's write path restores the store it saved, so a failed write leaves the board as it was" is false.
The `EnsureFailingRunSettingsCarry` row would then see the moved entry, and D7's acceptance criterion "a walk whose board write fails leaves the entry as it was" fails.
A commit or push failure in the detached sync also never reaches `carryMovedRunSettings`, which the card's skip logic assumes it does.

### [MEDIUM] Card 9 widens the walk to verbs that load no loom config

**Location:** `_lym/plan/09-reconcile-before-load.md:31`
**Issue:** The card places `ReconcileBeforeLoad` before the pre-run's skip of `loom`, `checklist` and `pusher` (`internal/loomcli/cli.go:268`).
D5 licenses the walk for verbs that load the loom or tapestry config, and the checklist, the pusher and the bare group load neither.
The pusher is the background process every run commit, dobby tick and resume starts.
It resolves its worktree from `--worktree` and starts in the hub's Bolt directory (`internal/loomcli/pusher.go:24-26`), so resolving "the location from cwd" there points at the Bolt.
On a stale stamp it would take the hub lock for up to `DefaultLockWait` and walk and commit in every worktree.
The card states no bound for this.

### [MEDIUM] hubreconcile/doc.go still says only a start verb runs the walk

**Location:** `internal/hubreconcile/doc.go:3`
**Issue:** Line 3 says "Its one entry point is Ensure; a start verb calls it, through EnsureOrWarn, before loading any module config."
Line 33 says "the next start verb retries".
After card 9 every verb that loads a reconciled config calls it, and card 8 adds the run_settings carry as a new skip cause.
Card 9 lists the file only under **Uses:**, and cards 6 and 8 only extend it, so no card rewrites those lines.

### [MEDIUM] PATTERN-config-layers' background keeps a rule card 6 breaks

**Location:** `_lym/plan/02-config-moved-keys.md:61`
**Issue:** `pattern/PATTERN-config-layers.md:7` says no step removes an unknown base-layer key except the operator's `lym config reconcile` or `--unset`.
Lines 77-78 say every other reconcile, the binary-change walk included, keeps such a key.
Card 6 makes the walk rename the table's old keys in hub base files, and drop them when null or when the new path is set.
Card 2 edits three other parts of the file, and card 14 changes key names only, so these two statements stay false.

### [LOW] Card 4 calls a changing classification test unchanged

**Location:** `_lym/plan/04-nest-loom-tapestry.md:121`
**Issue:** `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` table lists flat loom and tapestry keys (`internal/configreg/configreg_test.go:175-176`).
Once `LocalDenied` derives from the nested categories, those rows must move.
The test is already a target, so only the Intent's "unchanged" misleads.

### [LOW] Card 6's failure-row claim names a branch its rows never reach

**Location:** `_lym/plan/06-reconcile-moves.md:63`
**Issue:** The test's unparseable-file rows use `batcher`, `shuttle` and a hub file.
None of these modules has a move table, and the card says `moveOldKeys` returns such a document unchanged.
Their error still comes from the later prune or reconcile, and the move's own `*FileError` branch has no reaching row.

### [LOW] Seed-Child's single way forward is checked only against authored text

**Location:** `_lym/plan/05-moved-key-tables.md:50`
**Issue:** The row's fake `PrepareRunSettings` returns moved-key text the test writes itself.
D4's criterion that Seed-Child prints exactly one way forward depends on `refuseMovedKeys` printing none of its own, and this row never checks that composition.

### [LOW] Card 8's unreadable-board row is reached by the missing board config

**Location:** `_lym/plan/08-board-run-settings-carry.md:49`
**Issue:** `carryHub` writes only `board.json` (`internal/hubreconcile/carry_test.go:19-34`).
`RewriteRunSettings` opens the board through its board config, so the skip comes from the absent config, not from the unparseable board the row names.

### [LOW] Card 7's profiles write rule contradicts itself

**Location:** `_lym/plan/07-profiles-moves.md:32`
**Issue:** The first clause says a move alone is not a change when the follow writes the template's bytes.
The second says a write happens when the follow or the move changed the file.
Moving file and base together can leave the follow reporting no unit change while the on-disk bytes are still flat, so the file must be written on a byte difference.

## Rejected

- Lens affected-set's finding on card 10 and `ValidateMachine` tests: the `ValidateMachine` rows that assert refusal text are rows of `TestValidateLocal` (`internal/configengine/layers_test.go:591`, rows at `:645-652`), which card 10 already lists. Their substring assertions, such as `name` and `nope`, still match the full names.
- A research-step finding on cards 11, 13 and 14's run-time grep: a grep of `contracts/`, `pattern/` and `docs/` outside `docs/research/` for flat loom and tapestry names finds only files that cards 4, 13 and 14 already list. The re-grep is a completeness net, not a design fact left open.
- A run-lock refusal of card 8's rewrite of a running entry's `run_settings`: `scopeChanged` never compares `RunSettings` (`internal/boardengine/runlock.go:211-238`), so the board lock admits it.

## Focus departures

- "Planned tests across cards 1 to 10: settle whether any new test function or row duplicates coverage an existing test already gives, after reading that test in the code." I settled this for the new rows of existing tests, and for the new functions `TestKeyName`, `TestMoveKeys` and `TestMoveSettings`, which pin new public helpers no test covers. I did not read the hubreconcile integration file to compare `TestEnsure_MovesOldConfigKeysAndMailsTheMoves` and `TestEnsure_CarriesMovedRunSettingsOnTheBoard` row by row against its existing `TestEnsure_*` functions. I raised no finding on them.
