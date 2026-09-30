---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/04-send-launch-script-source.md
    summary: Card 4 misses ReplaceStrand's post-add SaveState failure, where the new strand's record can end up never persisted and its launch script orphaned
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/04-send-launch-script-source.md
    summary: No card tests the AddStrand SaveState-failure delete card 4 adds, so half the failed-add acceptance criterion is unexercised
  - id: F3
    severity: LOW
    location: _lyx/plan/04-send-launch-script-source.md
    summary: emptycmd_integration_test.go's file doc says the empty-Cmd send-keys literal is the composed prelude, which card 4 falsifies and no card updates
  - id: F4
    severity: NIT
    location: _lyx/plan/03-launch-script-cleanup.md
    summary: Card 3's ReplaceStrand test does not say how the fake survives the replacement strand's launch, which the default fake fails
---

### [MEDIUM] ReplaceStrand's post-add SaveState failure can orphan the new strand's script

**Location:** `_lyx/plan/04-send-launch-script-source.md`, "Failed add" bullets; `internal/reedengine/replace.go` `ReplaceStrand`.

**Issue:**
The decision record's rule is that "a launch that wrote a script but whose strand record is never persisted (a failed add) deletes that script", under the heading "Every path that drops strand records deletes their scripts."
Card 4 covers two failed-add paths: `addStrandLocked`'s launch failure and `AddStrand`'s post-add `SaveState` failure.
`ReplaceStrand` has a third.
When `addStrandLocked` succeeds, the script is written and the new strand sits in `st.Strands`;
`ReplaceStrand` then calls `SaveState`, and on failure only sets `addErr` without cutting the strand off the table.
The tail's `reconcileApplyPersistLocked(st)` runs next, and its `SaveState` is its last step (`spawn.go`), so:

- if `applyErr == nil`, the table including the new strand was persisted, and the script must stay;
- if `applyErr != nil`, its `SaveState` never succeeded either, so the new GUID was never persisted, and nothing ever deletes its script.

The second case is squarely the decision's "record never persisted" outcome: the next op loads a `reed.json` without that GUID, reaps the pane as untracked, and the script is orphaned permanently.
A single underlying cause (an unwritable `reed.json`) hits both saves, so this is not an exotic double fault.

**Fix:**
Add `internal/reedengine#Engine.ReplaceStrand` to card 4's Edit list.
Record the post-add `SaveState` failure separately from a launch failure, and after `reconcileApplyPersistLocked` returns, call `removeLaunchScripts` for the new strand's GUID when that save failed and `applyErr != nil`.
Add a test: a fake whose `split-window` returns a fresh pane id, and whose `Enter` send-keys replaces `reed.json` with a non-empty directory so every later `SaveState` fails its rename;
`ReplaceStrand` returns an error and the launch directory holds no script for the new GUID.

### [MEDIUM] The AddStrand SaveState-failure delete is untested

**Location:** `_lyx/plan/04-send-launch-script-source.md`, test list.

**Issue:**
Card 4 adds a delete when `AddStrand`'s `SaveState` right after `addStrandLocked` fails, but its only failed-add test drives `addStrandLocked` with a failing literal `send-keys`.
The acceptance criterion "a failed add leaves no script for its unpersisted GUID" therefore has one of its two paths (three, with F1) unexercised, and a regression dropping that delete would pass every test.
A test is cheap: `fsx.AtomicWriteBytes` ends in `os.Rename`, which fails when `reed.json` is a non-empty directory, and the fake's `Enter` send-keys hook runs after the script write and before the save.

**Fix:**
Add a test to card 4: `AddStrand` through the `e.tmux.execHook` fake, with `split-window` returning a fresh pane id and the `Enter` send-keys hook replacing `<e.stateDir()>/reed.json` with a non-empty directory;
`AddStrand` returns an error wrapping the persist failure and the launch directory holds no file.

### [LOW] emptycmd_integration_test.go's file doc goes stale

**Location:** `_lyx/plan/04-send-launch-script-source.md`, Edit list; `internal/reedengine/emptycmd_integration_test.go` file doc.

**Issue:**
The file doc states that "an empty-Cmd strand's launchStrandLocked send-keys literal is the composed prelude alone".
After card 4 the send-keys literal is the source statement and the prelude alone is the script's content.
The test body still passes, but the doc then describes behaviour the code no longer has, and no card lists the file.

**Fix:**
Add `internal/reedengine/emptycmd_integration_test.go` to card 4's Edit list, rewording the file doc so the prelude alone is the launch script's content and the send-keys literal is its source statement.

### [NIT] Card 3's ReplaceStrand test leaves the replacement launch unspecified

**Location:** `_lyx/plan/03-launch-script-cleanup.md`, test list.

**Issue:**
`ReplaceStrand` adds and launches a replacement strand.
The `execHook` fake the card models its tests on answers `split-window` with an empty string, which `validateSplitCreatedNewPane` rejects, so `ReplaceStrand` would return an error and the test would either fail or silently assert on an error path.

**Fix:**
Name the replacement spec's display as `render.AnchorHidden`, so `needsLaunchOnAdd` skips the launch and the test isolates the removal step's script deletion.

## Focus-directive verdicts

- **ReplaceStrand post-add `SaveState` failure:** covered by the decision's rule, and card 4 must name it — see F1.
  The record is never persisted only when `reconcileApplyPersistLocked` also fails; when it succeeds the record lands and the script must stay, so the delete is conditional on both.
- **Test coverage of the `AddStrand` `SaveState`-failure delete:** not exercised by any card — see F2.
- **Every drop path reached, no unlicensed delete:** cards 3 and 4 reach `removeStrandLocked` (behind `RemoveStrand` and `ReplaceStrand`), `Down`, and the `addStrandLocked`/`AddStrand` failed adds;
  the only gap is F1.
  No card deletes on `Resume`, `UpdateStrand`'s surface, or the `--if-absent` relaunch, whose records are already persisted.
  `removeStrandLocked` deletes scripts before the removal is persisted, so a later failed persist leaves a persisted GUID without a script;
  that is within the invariant (a script exists *only while* its GUID is persisted), and the next launch of that strand rewrites the file, so it is not a defect.
  `Down` returns early on `serverErr` before deleting `reed.json`, and card 3 places the directory removal after that delete, so the two stay consistent.
- **Granularity of cards 2–4:** not a defect.
  Card 2's three helpers are one independently testable unit (`launchScriptDir`/`launchScriptPath` are inseparable from `stageLaunchScript`) with its own tests;
  card 3 is one unit, the removal step plus `Down`;
  card 4 must carry the docs by the Documentation Lifecycle, and the failed-add deletes can only be tested once the chokepoint writes scripts, so bundling them with the wiring is sound.
- **`ImpactSummary` on cards 1–4:** each carries a conclusion the code bears out.
  `Shell`'s only implementers are `posixShell` and `pwshShell` (no other `Quote`/`ReadFile` implementations exist in the repo);
  selvage's split, `new-session`, and `io.go`'s `Send` never reach `launchStrandLocked` or `composePaneLaunchLine`.
  Card 4's summary gains `ReplaceStrand`'s post-add path once F1 is fixed.
