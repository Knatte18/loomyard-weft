---
verdict: BLOCKING
findings:
  - id: F11
    severity: BLOCKING
    class: design
    location: _lym/plan/09-reconcile-before-load.md:71
    summary: "Card 9's new tier1 test runs `lym shed step` on a seeded loom run with a free run lock, which goes on to `reed.Up()` (a tmux server) and then drives a producer, breaking PATTERN-test-speed's rule that tier1 has no tmux server."
    fix: "Rewrite the `TestShedVerbs_StaleStampReconcilesBeforeALoomRecipeArms` bullet in card 9. The test holds the seeded run's run lock before it runs `lym shed step`, so the verb arms through `loomcli.ArmAt` and its full `wire`, which loads loom's config strictly, and then refuses at `loomPreStep`'s early probe with kind busy, before the seed commit and `reed.Up()` (the shape of the `loom step` arm in `internal/shedcli/parity_test.go`). It asserts that the refusal is the busy one and not the moved-key one, and that the hub base `loom.yaml` holds `code.max_bounces: 2` afterwards."
  - id: F12
    severity: MEDIUM
    class: design
    location: _lym/plan/09-reconcile-before-load.md:36
    carries: F1
    summary: "Card 9 says its sites are every route to a strict loom or tapestry load but still misses the dobby: `lym beam dobby` reads `tapestry.implementer` through `DobbyRunNotices`, `readAgentActivity`, `gateProcessLive` and `fixerTimeout` (`configreg.SettingLookup`), and a dobby that re-execs onto a new binary runs no walk first, breaking D5."
    fix: "In card 9, add `internal/beamcli#hubDobbyDeps` to **Edit:**, and add a bullet saying that `hubDobbyDeps` calls `hubgeom.ReconcileBeforeLoad(prime)`, so every dobby start and re-exec walks before its first run notice reads `tapestry.implementer`. Name the dobby in the ImpactSummary. `validate-plan --overlay` with this target reports no new blocking finding."
  - id: F13
    severity: MEDIUM
    class: design
    location: _lym/plan/08-board-run-settings-carry.md:41
    carries: F2
    summary: "Card 8 restores `board.json` only on a failure at the hook. A render failure leaves the moved entries saved, so D7's 'a failed read or write ... leaves the board as it was' does not hold, the retry finds no old key and never renders again, and the orch is never mailed the moved entries."
    fix: "In card 8, make the restore cover every failure after the save, inside the write lock. Add `internal/boardengine#Board.boardCriticalSection` to **Edit:** and give it a rollback that it runs before releasing the lock when `afterSave` or the render returns an error; `RewriteRunSettings` passes the rollback that writes the saved `board.json` bytes back. Replace the sentences on lines 41 and 49 that leave the moved entries saved after a render failure, and add a row to `TestEnsure_CarriesMovedRunSettingsOnTheBoard`, or a tier0 row beside `TestCarryMovedRunSettings_UnreadableBoardIsSkipped`, in which the render fails (a directory at a rendered output's path) and `board.json` keeps its bytes. `validate-plan --overlay` with this target reports no new blocking finding."
prior:
  - id: F1
    status: open
    answer: "The shed, batten and merge-in routes it named are now covered. The method it asked for, every route to a strict load, still misses the dobby's run notices."
  - id: F2
    status: open
    answer: "The reason given for moving the snapshot and restore inside the lock holds. It does not answer the 'any later step' half: a render failure still leaves the board changed."
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
---

### [BLOCKING] The shed walk test needs a tmux server in tier1

**Location:** `_lym/plan/09-reconcile-before-load.md:71`

**Issue:** `loomPreStepUnmarked` (`internal/loomcli/arm.go`) probes the run lock, seeds and commits, and calls `c.reed.Up()`.
`shed.Step` then drives a producer.
`internal/shedcli/parity_test.go` states that its tier1 suite stays below tmux only by holding the run lock for the `step` arm.
The planned test holds no lock.
PATTERN-test-speed gives tier1 no tmux server, and `tmuxkit.Main` checks that at the end of the package.

### [MEDIUM] The dobby loads tapestry with no walk

**Location:** `_lym/plan/09-reconcile-before-load.md:36`

**Issue:** `internal/battencli/dobbynotice.go` wires `readAgentActivity`, which reaches `fixerTimeout`'s `configreg.SettingLookup` of `tapestry` for a live verify fixer.
`internal/beamcli/dobbydeps.go` builds those notices.
`lym beam start` runs the walk, but `internal/dobbyengine/doc.go` has the dobby re-exec onto a changed binary between ticks, with no start verb in between.
A pair's local `tapestry.yaml` that still holds an old key is then refused at this load.

### [MEDIUM] A render failure leaves the board changed

**Location:** `_lym/plan/08-board-run-settings-carry.md:41`

**Issue:** `boardCriticalSection` saves `board.json`, runs `afterSave`, then renders.
Card 8 restores only from inside `afterSave`, and says so.
D7 makes a failed write leave the board as it was and have the next walk retry it.
After a render failure the retry writes nothing, so the rendered views stay stale until another board write.

## Focus verdicts

- Card 9 routes: the pre-run, shed, batten, merge-in, `hubLandDeps`, `wireHub` and `loomConfigOf` sites hold; the dobby is the one route I found missing (F12).
- Card 9 skip order and bounds: holds. The pusher, checklist and bare group return before the walk, the pre-push check loads no loom config, and the Bound states the repeat-walk cost.
- Card 8 claims: the lock, the restore point, the detached sync and the line references match `internal/boardengine/board.go`. D7 is not met on a render failure (F13). `run_settings` is outside `scopeChanged`, so a held run does not refuse the carry.
- Planned rows: the Seed-Child text matches `RunSettingsError` as `prepareRunSettings` wraps it. The `loom.yaml` parse-failure row reaches a module with a table. The unreadable-board row, with a board config added, reaches the board read.
- Hubreconcile tests: neither new function duplicates an existing `TestEnsure_*` or a lower-tier row, since the mail, the commits and the board carry are covered nowhere else.
- Statements left false: the two named files are rewritten. Others are listed below.
- Card 4 and card 7: both hold. `FollowTemplate` returns its input byte for byte when no unit changes, so the byte comparison writes an untouched flat file.

## Not raised: outside this incremental round's changed hunks

These are real, and none can be a finding under the incremental rule.
The first would be BLOCKING in a full round.

- D4's `--unset` removal route fails for a file holding two old keys (cards 2 and 5). `setChecked` (`internal/configcli/configcli.go`) rolls a write back when the load error after it differs from the one before. Card 2 reports the first old key in sorted order, so unsetting that key changes the error to name the next one, and the removal is undone. At the hub default layer, `setHubWide` runs `validateHubFile` and rolls back while any old key remains. Card 5's `TestDispatchSet` row holds one old key, so it passes. A fix would add `internal/configcli#setChecked`, `internal/configcli#setHubWide` and `internal/configcli#TestDispatchLayers` to card 5, with `internal/configcli/doc.go` under **Uses:**; `validate-plan --overlay` admitted that set.
- Card 9's `TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads` names `lym loom circling`, which only lists subcommands and loads nothing. Only `circling continue` reads caps, on an escalated run, and it reads the hard cap, not `max_bounces`.
- Card 8's "a second walk writes no board commit" cannot fail: under `go test` the detached sync's `selfexec.LymBinary` returns `ErrTestBinary`, so the carry never commits. Asserting unchanged `board.json` bytes and no second notice would check it.
- Card 11's target list leaves out packages with bare-key messages, such as `internal/sheddriver`, `internal/lenses` and `internal/weaverengine`. It also leaves out `seatengine`'s `TestEngine_Run_AdvisorStartFailureIsRecordedAndNamedToTheChair`, which asserts "in discussion_advisors" and breaks once the name is rendered in full.
- Card 6 collects the moved lines and mails them when the walk ends, so a pair failing after the hub-wide commit loses the notice for keys already moved.
- Still false after the plan: the `lym config reconcile` help in `internal/configcli/configcli.go` (no move, and "the one way to remove a key the template lacks"), and "start verb" statements in `repoDefaultAbsent`, `internal/preflightshed/doc.go`, `internal/fabriccli/clone.go`, `docs/overview.md` and `docs/shared-libs/configengine.md`.
