---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-retire-shell-window-stack.md:115
    summary: "Card 2 removes the size refusal from ValidateDisplay but leaves TestUpdateStrandLocked unlisted, so its TopPositionalDisplayRejected row (expects the content-sized error) fails at card 2's commit"
    fix: "In 02-retire-shell-window-stack.md add `internal/reedengine#TestUpdateStrandLocked` to the Edit list (after `internal/reedengine#TestWindowOrder_RealTmux`) and add one Intent sentence in the Tests paragraph: TestUpdateStrandLocked drops its TopPositionalDisplayRejected row, which pinned the content-sized refusal card 2 removes, and keeps every other row for card 5. Checked with `lyx loom validate-plan --overlay`: the combined run of this and the other target-changing fixes reports no finding beyond the baseline."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/02-retire-shell-window-stack.md:156
    summary: "The plan rewrites file headers and doc comments only where a card remembers them, and the third verify grep exempts every test file in four packages, so headers describing deleted members stay stale: windowsize.go, watchdog.go, attach.go, state.go, bar.go, ownwindow.go and three test headers"
    fix: "Add these file self glyphs to card 8's Prosa list and one Intent sentence saying each header is rewritten to describe only what its file still holds after cards 1 to 7: `internal/reedengine/windowsize.go` (readbacks, title-row readback and hook array gone, pins and unpin stay), `internal/reedengine/watchdog.go` (resize signal file, hook command and loop timings gone), `internal/reedengine/attach.go` (readbacks, layout guards and layout plan gone), `internal/reedengine/state.go` (toRenderStrands gone, so the layout-decision sentence goes), `internal/reedengine/bar.go` (pane-border title gone), `internal/reedengine/ownwindow.go` (no window 0 layout to contrast with), `internal/reedengine/windowsize_test.go` (only the geometry pin test remains), `internal/reedengine/bar_integration_test.go` (border-title half gone), `internal/reedengine/strandops_integration_test.go` (collapsed_rows stack sentence). Checked with `lyx loom validate-plan --overlay` together with the F1 and F3 fixes: no new blocking finding."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-retire-pane-era-anchors.md:67
    summary: "Card 5 lists TestMultiplexerContract under Edit but its Intent names no change to it; the test holds no Display or anchor, and the only stale text is two comments, one naming launchStrandLocked's split path and one citing apply_test.go and render.Rules that card 2 deletes"
    fix: "Add `internal/reedengine#TestMultiplexerContract` to card 2's Edit list (after `internal/reedengine#TestWindowOrder_RealTmux`) and say in card 2's Tests paragraph that its step (c) comment stops citing apply_test.go and render.Rules; keep the card 5 Edit entry and add one Intent sentence there saying its step (b) comment stops describing the raw split-window as the path launchStrandLocked uses. Checked with `lyx loom validate-plan --overlay` together with the F1 and F2 fixes: no new blocking finding."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/08-sweep-docs-and-sandbox.md:25
    summary: "Card 8 drops the stacked-adds scenario from the reedcli smoke headers, but no card retires or renames the StackedAddsKeepEverySessionPane subtest, whose comment still says below-parent strands and split-window"
    fix: "In card 5's Tests paragraph state that the TestSmokeLifecycle subtest StackedAddsKeepEverySessionPane is renamed AddsKeepEverySessionPane and its comment rewritten to say that successive default adds each open a window of their own and yield one live pane per strand plus Selvage in the session-wide listing; in card 8 change the two smoke header edits from dropping the scenario to rewording it as one live pane per strand across successive adds, since the subtest stays."
---

### [MEDIUM] Card 2 breaks TestUpdateStrandLocked before card 5 reaches it

**Location:** `_lyx/plan/02-retire-shell-window-stack.md`, Edit list and Tests paragraph; `internal/reedengine/strand_test.go:262`.

**Issue:** Card 2's Intent makes `ValidateDisplay` return nil for every anchor but own-window, with no size branch.
`TestUpdateStrandLocked` has a row `TopPositionalDisplayRejected` that updates a strand to `render.Display{Anchor: render.AnchorTop}` and wants an error containing `content-sized`.
That error comes from the `SizePositional` branch of `ValidateDisplay` in `render/policy.go`, which card 2 deletes.
`validateDisplay` in `strand.go` still accepts `top` until card 5, and `refuseWindowMove` only guards own-window and overview moves, so after card 2 the update succeeds and the row fails with "want rejection".
Card 2 lists `TestAddStrandLocked` for its size rows but not `TestUpdateStrandLocked`; card 5 does edit it, but for the anchor move, one card too late.
Every other test that matched the size refusal messages is either deleted by card 2 or listed there (checked by grep for `content-sized`, `unknown size kind` and `min rows`).
The overview says every card builds; a failing unit test at card 2's commit breaks any per-card test gate.

### [LOW] Stale file headers and comments are left by a method that relies on recall

**Location:** `_lyx/plan/02-retire-shell-window-stack.md` (the headers it leaves behind); `_lyx/plan/00-overview.md:39` (the third verify grep).

**Issue:** The third `## verify:` grep exempts every `*_test.go` in `internal/reedengine`, `internal/reedengine/render`, `internal/reedcli` and `internal/shuttlecli`, so no check catches stale stack wording there, and for non-test files the plan fixes only the headers a card names.
After cards 1 to 7 these headers describe deleted members and nothing rewrites them:
- `windowsize.go` lines 1 to 4: the window-size query, effective-value readbacks, title-row readback and the `window-resized` array write side, which card 2 deletes.
- `watchdog.go` lines 1 to 4: the resize signal file location and hook command string, which card 2 deletes (card 1 drops only the loop timings).
- `attach.go` lines 1 to 4: readbacks, layout guards and layout plan, which card 2's always-bare argv removes.
- `state.go` lines 2 and 5: `toRenderStrands`, deleted by card 2.
- `bar.go` line 1: the strand pane-border title, whose `paneBorderFormat` card 2 deletes (card 3 edits only line 1's button order).
- `ownwindow.go` line 3: "window 0's layout never places it", which presumes a layout.
- `windowsize_test.go` lines 1 to 3, `bar_integration_test.go` line 3 and `strandops_integration_test.go` line 4 (the `collapsed_rows` stack sentence).
Every grep-scanned non-exempt file that names `below-parent` is covered: `shuttleengine/attach.go:406` sits in `normalizeAttachSpec`, `selvagepane.go:425` is the doc of `shellWindowOf`, `render/policy.go` is rewritten by card 2, and `shuttlecli/run.go`, `reedcli/add.go`, `docs/overview.md` and `CLAUDE.md` have cards.
The gap is the headers that name no grep token.
D8 and CLAUDE.md's docs-land-in-the-same-commit rule ask for the sweep, and the plan's own `render/policy.go` and `watchdog_test.go` header edits show the intent; the finding is the method, so it is one finding and not one per file.

### [LOW] Card 5 lists TestMultiplexerContract with no stated change

**Location:** `_lyx/plan/05-retire-pane-era-anchors.md`, Edit list; `internal/reedengine/contract_integration_test.go:68`.

**Issue:** `TestMultiplexerContract` drives raw tmux and builds no `Display`, strand or `AddStrand` call, so card 5's "moves to own-window" sentence does not apply to it.
The only text that goes stale is the step (b) comment "the tallest-pane split path launchStrandLocked uses", which card 5 makes false, and the step (c) comment citing `apply_test.go` and `render.Rules`, which card 2 deletes.
The Intent names neither, so the target is an Edit with no described edit.
The other card 5 Edit tests were each checked against the `AnchorBelowParent` and `AnchorTop` sites by enclosing function, against the `--anchor` default their reedcli smokes rely on, or against a named change.

### [LOW] StackedAddsKeepEverySessionPane outlives the header line card 8 removes

**Location:** `_lyx/plan/08-sweep-docs-and-sandbox.md`, last Prosa paragraph; `internal/reedcli/smoke_lifecycle_test.go:83`.

**Issue:** Card 8 drops "layout survival under stacked below-parent adds" from the headers of `smoke_test.go` and `smoke_lifecycle_test.go`.
The subtest that scenario names, `StackedAddsKeepEverySessionPane` in `TestSmokeLifecycle`, stays: card 5 lists `TestSmokeLifecycle` and `listPaneLines` and makes the adds open own windows, so the subtest still asserts one live pane per strand plus Selvage.
Its comment still says "below-parent strands" and "session-target split-window", and no card rewrites it, while the header no longer lists the scenario.
The file is exempt from the third verify grep, so nothing catches the mismatch.

## Rejected

- Compile order across cards 2 and 5: every test member that references `render.AnchorTop` or `render.AnchorBelowParent` (found by grep, mapped to its enclosing function) is deleted or listed by a card at or before card 5, and each test member that uses `render.Size`, `Box`, `Params`, `Rules`, `Strand` or `Pin` is deleted or listed by card 2.
  Helpers defined in whole-file deletions (`liveRenderStrands`, `planCollapsedRows`, the `zoom_integration_test.go` and dot-fill helpers) have no user outside the deleted files.
- Stale stack wording in grep-scanned files: all seven non-exempt files the third verify grep matches today are rewritten by a card; the remaining gap is F2.
- Emptied files: `windowsize_test.go`, `watchdog_test.go`, `selvagepane_test.go`, `attach_test.go` and `attachgeometry_integration_test.go` keep tests after card 2's deletions.
  `watchloop_test.go` is emptied by card 1 except `countLogLines`, and the card's Intent removes the emptied file; its file glyph cannot sit beside the member glyph on one card (`redundant-file-target`), so the Intent prose is the only admissible carrier and this is not a finding.
- The `render` test package keeps `types_test.go` and `testmain_test.go` after card 2, which compile without the deleted layout files.
