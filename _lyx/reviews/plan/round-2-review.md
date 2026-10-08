---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/15-vscode-keybindings-block.md:7
    summary: "Card 15 changes Spawn to return two values but omits internal/ideengine/menu.go, whose Menu calls Spawn in the one-value form, so go build breaks"
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/14-orch-color.md:36
    summary: "Card 14 adds orchengine watcher tests without listing cmd/lyx/testdata/test-budget.yaml, and the orchengine row already equals its count, so the budget test fails go test"
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-two-line-status-bar.md:3
    summary: "Existing tmux-tier tests assert a one-row status bar (window height rows-1); status 2 breaks them, watchdog_integration_test.go is in no card, and attachgeometry_integration_test.go is only in card 8"
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-reed-switch-verb.md:10
    summary: "Sandbox mission M19 still requires the retired status-left identity text, and no card that retires it lists the mission; yamlengine.md still cites status_line as its example"
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/02-reed-segment-color-config.md:42
    summary: "Card 2 plans validateSegmentColors rows in a new test file for a boot message that its own TestUp_BootValidation row already asserts"
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/plan/04-reed-switch-verb.md:43
    summary: "Card 4 documents switch-client as an optional probe verb, but the optional-verb list is created only by card 7"
---

### [BLOCKING] Card 15 leaves `ideengine.Menu` calling the old `Spawn` signature

**Location:** `_lyx/plan/15-vscode-keybindings-block.md:7` (the Edit list), `:36` and `:43` (the Spawn return change)

**Issue:** Card 15 makes `ideengine.Spawn` return the keybindings result beside its error.
It lists `internal/ideengine/spawn.go`, `internal/idecli/cli.go` and the two `integration` test files as the callers to update.
`internal/ideengine/menu.go:109` also calls it: `if err := Spawn(l, chosenSlug); err != nil {`.
That is a one-value assignment from a two-value call, so the untagged `go build ./...` fails the moment the card lands.
The writer stencil requires every card to build on its own.
A grep of the whole repository for `Spawn(` finds only `menu.go`, `spawn.go`, `idecli/cli.go` and the two integration files, so `menu.go` is the one missed caller.
`menu_test.go` reaches `Spawn` through `Menu`, and the `KeybindingsPath` seam set in `testmain_test.go` already covers it, so no test file is missing.

**Fix:** Add `internal/ideengine/menu.go` to card 15's `**Edit:**` list, and state in the Intent that `Menu` takes the new return and discards or logs the outcome.

### [BLOCKING] Card 14 adds `orchengine` tests with no budget row

**Location:** `_lyx/plan/14-orch-color.md:36` (Tests), against the Edit list at `:3`

**Issue:** `cmd/lyx/testbudget_test.go` fails when a package's top-level `func Test` count exceeds its row in `cmd/lyx/testdata/test-budget.yaml`.
The `internal/orchengine` row is 122 and the package holds exactly 122 top-level tests, counted with the same rule as the budget test.
Card 14 says `watcher_test.go` "gains rows" for four distinct behaviors: the color types on the first idle tick and only once, the reload types it first, a failing `TypeColor` still advances the reload, and a restart mid-step re-types it.
`watcher_test.go` has no table that fits them; it holds one top-level `TestWatcher_*` function per behavior, such as `TestWatcher_ClearCycleReloadsSkillsThenPointer` and `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep`.
Four new behaviors there become new functions, which puts `orchengine` over budget.
Card 14 lists no `test-budget.yaml` target, although the plan's own Shared Decision "Test budget" requires one of every card that changes a package's count, and the plan-level `go test ./...` runs the budget test.
Cards 12 and 13 avoid the problem by stating "no new test function"; card 14 states neither.

**Fix:** Either add `cmd/lyx/testdata/test-budget.yaml` to the Edit list with the `internal/orchengine` row set to the count after the card, or name the existing tests whose assertions grow so that no function is added.

### [MEDIUM] Existing tmux-tier tests assert a one-row status bar that card 5 replaces with two rows

**Location:** `_lyx/plan/05-two-line-status-bar.md:3` (the Edit list), `_lyx/plan/08-title-row-geometry.md:3`

**Issue:** Card 5 pins `status 2`, so the live window height becomes the client's rows minus 2.
Three tmux-tier tests hard-code the one-row case:

- `internal/reedengine/watchdog_integration_test.go` asserts `h != newRows-1` at line 151, `h != finalRows-1` at line 220 and `h != 41` for a 42-row client at line 254.
- `internal/reedengine/attachgeometry_integration_test.go` asserts `gotH != rows-1` at line 218, and `resizedRows-1` at lines 425 and 518.

No card lists `watchdog_integration_test.go`.
Card 8 lists `attachgeometry_integration_test.go` but only to "gain the D7 acceptance rows", and the break begins at card 5, three cards earlier.
Acceptance requires the tmux-tier tests to pass, and Publish runs that tier.
A card executor holds only the files its card names, so the plan as written cannot fix `watchdog_integration_test.go`.
`go vet -tags tmux` compiles these tests and so cannot see the break.

**Fix:** List both files in card 5, and state in its Intent that the expected heights now read the reserved rows from `#{status}`, or are `rows-2`.

### [LOW] Stale references to the retired status line outside any card's targets

**Location:** `_lyx/plan/04-reed-switch-verb.md:10` (it edits `tools/sandbox/SANDBOX-REED-SUITE.md` and adds a mission only), `_lyx/plan/06-retire-status-line-pipeline.md:25`

**Issue:** Mission M19 of `tools/sandbox/SANDBOX-REED-SUITE.md` requires that `tmux display-message -p '#{status-left}'` show the rendered identity text, and makes a blank readback a `FAIL`.
Cards 5 and 6 remove the `status-left` pin and the text, so an operator running M19 after the plan lands gets a false `FAIL`.
Card 4 edits that file to add one mission and leaves M19 as it is.
`docs/shared-libs/yamlengine.md:116` also cites `status_line` as its example of a whole missing mapping; card 6 retires that key and lists no edit to the file.
The writer stencil asks for a grep of the repository for retired text, with every file asserting it joining the card's targets.

**Fix:** Add `tools/sandbox/SANDBOX-REED-SUITE.md` to card 5, rewriting M19's identity-text check to the two-line bar.
Add `docs/shared-libs/yamlengine.md` to card 6, swapping its example key.

### [LOW] Card 2 tests `validateSegmentColors` in a new file for a message a boot-validation row already asserts

**Location:** `_lyx/plan/02-reed-segment-color-config.md:42` and `:44`

**Issue:** Card 2 plans a new `segmentcolor_test.go` whose table includes `validateSegmentColors` rows, among them a bad value whose error names the key, value and palette.
The same card adds a row to `TestUp_BootValidation` (`internal/reedengine/lifecycle_test.go:33`) with `segment_colors.review: crimson`, asserting that `Up` fails naming the key, the value and the palette.
`validateSegmentColors` is an unexported helper whose only caller is `Up`'s validation block, and `PATTERN-test-economy` has an unexported helper tested through the surface that uses it.
The all-valid case is covered by every existing `Up` test that boots with the template defaults.

**Fix:** Drop the `validateSegmentColors` rows and keep the `TestUp_BootValidation` row.
The `segmentColor` table stays.

### [NIT] Card 4 documents an optional probe verb that card 7 introduces

**Location:** `_lyx/plan/04-reed-switch-verb.md:43`

**Issue:** Card 4's `doc.go` text adds `switch-client` "to the subcommand set as an optional verb".
`probe.go` holds only `requiredSubcommands` today, and the optional-verb list that makes the word true is created in card 7.
Between those two cards the documentation describes a mechanism that does not exist.

**Fix:** In card 4, say only that `SwitchClient` issues `switch-client`, and leave the optional-verb wording to card 7's `doc.go` paragraph.

## Focus verdicts

Each directive was judged afresh; none was departed from.

- **Test budget.** F2 settles it for `orchengine`.
  Cards 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 and 15 each list `test-budget.yaml` wherever they add or delete top-level tests, and name each row they set (`reedengine`, `vscode`, `hubgeom`, `claudeengine`, `shuttleengine`, `reedcli`, the new `segmentcolor`, and the dropped `tokenvocab`).
  Cards 12 and 13 add no function and say so.
- **Tagged-tier compile coverage.**
  Card 6 names every file that sets or reads `reedengine.Geometry.RepoName`: the three `tmux` files `ensuresession_integration_test.go`, `mouse_boot_integration_test.go` and `contract_integration_test.go`, the untagged `lock_test.go`, both tellers and their tests, and the deleted `statusline_test.go` files.
  Every remaining `RepoName` hit is `lyxcwd.Location.RepoName`, which stays.
  The `StatusLine` config field, `StatusLineText`, `ValidateStatusLine` and the `tokenvocab` importers are all covered by cards 5 and 6.
  Card 15 does not do so: F1 is the missed caller.
  Cards 11 and 14 name every implementer of the `Engine` and `Session` interfaces they extend.
- **Attack surface of stored tmux options.**
  Card 2 states that only palette names leave `segmentColor`, card 3 gives the writer, value and unset behavior of `@strand_color`, and card 7 bounds the click-time `@lyx_strands` expansion to a tmux-issued window id and states that no marker expands to nothing.
  Card 9's `@lyx_rezoom` is written only by the hook's own record entry and tested only for truthiness.
  I checked on tmux 3.6 whether a stale value could survive the re-zoom entry; `resize-pane -Z` on a one-pane window does not error, the list is not aborted, and the option is cleared.
  I found no reachable stale path, so this is not a defect.
- **Ordering sources.**
  Card 10 orders pairs by the modification time of the pair's `seed.json`.
  `shedrun.WriteSeed` is a no-op against an agreeing existing seed, so the file's mtime is written once, and the card and the plan's Shared Decisions both state that a checkout, rebase or re-materialization resets it and how far that misorders.
  Not a defect.
