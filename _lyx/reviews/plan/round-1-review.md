---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/07-reed-own-window-bar.md:36
    summary: >-
      Card 7 pins a new set-hook in pinGeometryOptionsLocked, which the attach pre-flight runs before its suppressing guards,
      so it breaks TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv and TestAttachArgv_PreflightOnAKnownGoodSession, and no card targets either test.
    fix: >-
      In card 7, add `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and
      `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession` to `**Edit:**`, and add a Tests bullet for each.
      For TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv, replace `fake.mustNotCall("set-hook")` with an assertion that no
      `set-hook` call names `windowResizedHookName`, since the degraded rows still reach the pin.
      Its doc comment changes from "issue no set-hook call at all" to "install no resize-pin hook".
      For TestAttachArgv_PreflightOnAKnownGoodSession, filter the `set-hook` calls to `windowResizedHookName` before the
      first-index and `-u` checks, and assert exactly one `session-window-changed` call carrying `unseenClearHookCommand`.
      Checked with `lyx loom validate-plan --overlay` together with F2 and F6; no new finding.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-render-own-window-display.md:31
    summary: >-
      Card 1 makes render accept own-window, which breaks the PlanError_DeferredAnchorRejected row of TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv.
      Until card 3, anyPlacedStrand still counts the row's own-window strand, so render plans an empty-cell layout and the pre-flight installs the resize hook.
      No card targets the row.
    fix: >-
      In card 1, add `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` to `**Edit:**`.
      Add a Tests bullet that renames the row `PlanError_DeferredAnchorRejected` to `PlanError_TopAnchorWithoutContentSizeRejected`,
      with the strand `{GUID: "a", PaneID: "%1", Display: render.Display{Anchor: render.AnchorTop, Size: render.Size{Kind: render.SizePositional}}}`.
      render.ValidateDisplay still refuses that display, so the row keeps pinning the plan-error path.
      Correct the ImpactSummary's Render-only claim to name the reedengine attach row.
      An own-window strand never being placed is already pinned by card 3's TestAnyPlacedStrand row.
      Checked with `lyx loom validate-plan --overlay` together with F1 and F6; no new finding.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-reed-own-window-launch.md:46
    summary: >-
      Card 4's relaunch kill of a strand's dead own-window pane is a kill D6 rules out, since D6 allows one only on removal.
      It can end the window without D5's viewer select, and card 5 then places the strand from a listing taken before the kill.
      In a shared window, joinGroupWindowLocked can then split a pane that is gone, or groupLiveCount can return a window that is gone, so the relaunch fails.
    fix: >-
      In card 4's launchStrandLocked bullet, keep the kill but state it in the Intent as a deliberate extension of D6.
      A dead pane left behind in a shared window would never be killed or ended, because reconcile kills no own-window pane and ends no window holding a bound pane.
      Bound it to the strand's own recorded pane on a relaunch.
      Before the kill, when that pane is the last listed pane of its window, run `selectStrandsWindowIfCurrentLocked(st, []string{<its window id>})`, so a viewer on it lands on window 0 (D5's bound).
      In card 5's own-window branch, before `groupLiveCount`, re-list the session's panes through `tmux.listWindowPanes(e.SessionName())`.
      Do this after the dead-pane kill, and also after any head reconcile that ended an own window, which card 3 keeps out of `killed`.
      The placement and `joinGroupWindowLocked` then read the current windows.
      Add a `TestOwnWindow_RealTmux` subtest to card 5, `RelaunchOfADeadGroupMemberRejoinsItsWindow`.
      In it, the first strand of a live group of two goes dead and `Resume` relaunches it into the group window beside its live partner.
      If its partner was removed first, the relaunch opens a fresh shared window instead.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/04-reed-own-window-launch.md:55
    summary: >-
      Card 4 prescribes a guard in openOwnWindowLocked that refuses an empty pane id or window id in the new-window answer,
      and it names no test that reaches the guard.
    fix: >-
      In card 4's Tests, add a fake-tmux row to `TestAddStrandLocked`, which card 4 already targets: `OwnWindowEmptyNewWindowAnswerIsRefused`.
      There, `new-window` answers an empty line for an own-window add, the add returns the error, and no strand is recorded.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/12-claude-open-launch.md:36
    summary: >-
      The resume-to-fresh fallback built with OrElse is a new routing edge, and card 12 states neither what it lets through nor what bounds it.
      The fresh line runs after any nonzero exit of the resumed session, not only after a failed resume.
    fix: >-
      Add a Bound sentence to card 12's Intent, after the fallback explanation.
      It says the fresh line runs whenever the resume line exits nonzero, including a resumed session that ends nonzero after a working life.
      It also says what bounds that: a fresh start under a session id whose transcript exists fails at once.
      The chain then goes on to the exit marker and the self-removal, so the fallback never starts a second live session over a recorded one.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/01-render-own-window-display.md:22
    summary: >-
      Card 1 rewrites the render package comment in types.go, and card 7 rewrites the file comment in bar.go.
      Both comments lie outside every member span either card targets, and adding the file beside its members is refused by redundant-file-target.
    fix: >-
      In card 1, replace the three `**Edit:**` glyphs `internal/reedengine/render#Display`, `internal/reedengine/render#Anchor` and
      `internal/reedengine/render#AnchorOwnWindow` with the file path `internal/reedengine/render/types.go`.
      In card 7, replace `internal/reedengine#statusFormatButtons` with `internal/reedengine/bar.go`.
      Neither comment lies inside a member span, so the file self glyph is the target that covers it.
      Checked with `lyx loom validate-plan --overlay` together with F1 and F2; no new finding.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/13-shuttle-open-verb.md:74
    summary: >-
      docs/overview.md line 667 also lists the shuttle verbs as run, interrupt, send, state and await,
      and card 13 updates only the bullet at line 375, so the second list goes stale without open.
    fix: >-
      In card 13's docs/overview.md bullet, also update the CLI sentence at line 667,
      changing `lyx shuttle run|interrupt|send|state|await` to `lyx shuttle run|open|interrupt|send|state|await`.
---

### [BLOCKING] Card 7's unseen-clear hook breaks two attach pre-flight tests it does not target

**Location:** `_lyx/plan/07-reed-own-window-bar.md:36`

**Issue:** Card 7 has `pinGeometryOptionsLocked` pin `set-hook -t =<session> session-window-changed ...` everywhere but Windows.
`AttachArgv` calls `pinGeometryOptionsLocked` at `internal/reedengine/attach.go:137`, before the `len(live) < 2`, `anyPlacedStrand` and plan-error guards that suppress the chain.
`TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` (`internal/reedengine/attach_test.go:224`) calls `fake.mustNotCall("set-hook")` on every row.
Its `FewerThanTwoLivePanes`, `NoStrandOwnsAPresentPane`, `ListPanesErrors` and plan-error rows all reach the pin, so they fail.
The fixture's `Watchdog: "on"` (`lock_test.go:68`) is why the pin issues no `set-hook -u` today.
`TestAttachArgv_PreflightOnAKnownGoodSession` asserts that the first `set-hook` call carries `-u` (`attach_test.go:318`).
The new hook comes from `pinGeometryOptionsLocked`, ahead of `installResizePinsLocked`'s clear, and carries no `-u`.
Card 7 edits only `TestPinBootOptionsLocked_InstallsTheSignalEntry` and the bar test.
The plan template requires every test a card's change breaks to join that card's targets.

### [MEDIUM] Card 1 breaks the attach plan-error row whose premise it removes

**Location:** `_lyx/plan/01-render-own-window-display.md:31`

**Issue:** The row `PlanError_DeferredAnchorRejected` (`attach_test.go:240`) uses an own-window strand on `%1` to reach the plan-error exit.
That premise goes false once card 1 makes `render.ValidateDisplay` accept own-window.
Between card 1 and card 3, `anyPlacedStrand` (`apply.go:133`) still counts the strand, because its pane is present in the window 0 listing.
`planCells` returns a plan with no placements and no band (`rules.go:52`), so `Rules` returns an empty-cell layout with no error.
The pre-flight then runs `installResizePinsLocked`, which issues `set-hook`, and the row fails.
After card 3 the row passes again, but now through the `anyPlacedStrand` skip rather than a plan error, so its name misstates what it pins.
No card lists the test, and card 1's ImpactSummary says the change is render-only.

### [MEDIUM] Card 4's relaunch kill is unlicensed by D6, skips D5's select, and leaves card 5 placing from a stale listing

**Location:** `_lyx/plan/04-reed-own-window-launch.md:46`

**Issue:** D6 says reed kills an own-window pane only when it is a recorded strand's pane being removed.
Card 4 adds a second kill, of a relaunching strand's dead pane, and records no deviation.
Without the kill, a dead pane in a shared window would stay forever, since reconcile kills no own-window pane and ends no window that holds a bound pane.
So the kill is needed, but it is undeclared and unbounded against D5.
When the killed pane is its window's last, the window ends with no `selectStrandsWindowIfCurrentLocked`, and tmux moves a viewer to whichever window it picks.
Card 5's branch then runs "after card 4's dead-pane kill" (`05-reed-window-groups.md:54`) on the `windows` listed before that kill.
Card 3 keeps reconcile's ended own windows out of `killed` (`03-reed-own-window-presence.md:92`), so a window that ended during the reconcile is not re-listed either.
`groupLiveCount` can then return a group window that is gone, and `joinGroupWindowLocked` splits "the group window's first listed pane", which can be the pane just killed.
`split-window -t <gone pane>` fails, and with it a live-server `Resume` or an `--if-absent` relaunch of a dead group member.
A seat table's chair, which is usually the first listed pane, is the common case.

### [LOW] openOwnWindowLocked's empty-answer guard has no named test

**Location:** `_lyx/plan/04-reed-own-window-launch.md:55`

**Issue:** The card prescribes an error when `new-window` answers an empty pane id or window id.
Neither card 4's Tests nor any other card's names a test that reaches that branch.
The rubric requires a named input and a named test for every prescribed guard.

### [LOW] Card 12's OrElse resume fallback states no bound

**Location:** `_lyx/plan/12-claude-open-launch.md:36`

**Issue:** `ResumeCmd: sh.OrElse(resumeLine, fresh)` runs the fresh `--session-id` line whenever the resume line exits nonzero.
That includes a resumed session that ran for a while and then ended nonzero, not only a `--resume` that could not start.
The Intent explains why a fresh start is needed, but it does not state this wider trigger or what keeps it safe, as the attack-surface rule requires for a new routing edge.
What keeps it safe is that a fresh start fails when the id already has a transcript, after which the chain goes on to the exit marker and the self-removal.

### [LOW] Two comment rewrites lie outside the cards' targets

**Location:** `_lyx/plan/01-render-own-window-display.md:22`

**Issue:** Card 1 rewrites the package comment above `package render` (`internal/reedengine/render/types.go:14`).
Card 7 (`07-reed-own-window-bar.md:27`) rewrites `bar.go`'s file comment (`internal/reedengine/bar.go:1-4`).
Neither comment lies inside a targeted member's span, and listing the file beside its members is refused by `redundant-file-target`.
The admissible target is therefore the file self glyph in place of the members it would hold.

### [LOW] A second shuttle verb list in docs/overview.md is left stale

**Location:** `_lyx/plan/13-shuttle-open-verb.md:74`

**Issue:** `docs/overview.md:667` lists `lyx shuttle run|interrupt|send|state|await`.
Card 13 already targets the file, but it updates only the bullet at line 375, so that list goes stale without `open`.

## Rejected

- Card 13 keeps `lyx shuttle open` on the shuttle group's existing `PersistentPreRunE` instead of `internal/cliwire`, which the decision record's constraint names.
  `PATTERN-cliwire-sole-wiring` binds the standalone-capable CLIs, and no shuttle verb uses `cliwire` today: only `burlercli`, `gatecli`, `loomcli`, `pausecli`, `reedcli` and `webstercli` import it.
  The card records the deviation with its reason, so this is not a PATTERN violation and needs no fix.
- Card 11's `TestShell_OrElse` and `TestShell_Exit`, one test per method, follow `internal/shell/shell_test.go`'s convention of one test per `Shell` method.
  `Shell` is the package's public surface, and each method is its own behavior, so this is not the per-symbol case the test-economy rule targets.
- `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` (card 3) pins the new `Present` and `OwnWindow` fields, which no existing `Status` test reads, so it is not redundant with `TestStatus_ReportsSegmentColor`.
- Card 9's list of `shuttleengine.Spec` sites leaves out the arbiter (`internal/loomcli/arbiterprompt.go`), but the arbiter ends with its row and needs no code change, so the omission has no effect.
- Fidelity: every Decision D1–D12 and every Constraint has a carrying card: D1 cards 1 and 4; D2 card 5; D3 cards 2, 5 and 6; D4 cards 4 and 7; D5 cards 3, 6 and 8; D6 card 3; D7 cards 4 and 5; D8 cards 3 and 4; D9 card 4; D10 card 9; D11 card 10; D12 cards 11–13.
  Cards 11–13 are licensed by D12, and card 10's window groups by D11.
- Card order: card 8 reads the fields card 3 adds, card 9 needs reed to accept own-window (card 4), and card 13 needs cards 4, 6, 11 and 12; each producer comes earlier, so no card depends on a later one.
- Verify: `## verify:` runs `./...` in the template's order, compiling the tmux and llm tiers and running the untagged and integration tiers, so it covers every targeted package.
