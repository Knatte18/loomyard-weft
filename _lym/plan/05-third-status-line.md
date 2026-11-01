# Card 5 — third-status-line

**Create:**
- `plan:internal/reedengine#buttonRowPlainOption` -> `const buttonRowPlainOption = "@lym_buttons_plain"`
- `plan:internal/reedengine#buttonRowAgentOption` -> `const buttonRowAgentOption = "@lym_buttons_agent"`
- `plan:internal/reedengine#statusFormatActions` -> `const statusFormatActions = listPrologueFormat + followButtonFormat`
- `plan:internal/reedengine#statusFormatPinArgvs` -> `func statusFormatPinArgvs() [][]string`

**Edit:**
- `internal/reedengine#statusFormatSessions`
- `internal/reedengine#followButtonFormat`
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine#Engine.repinStaleBindingsLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestBindingsBuildStamp`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`
- `internal/reedengine#TestStatusFormats_SessionLoopsTakeOneFormat`
- `internal/reedengine#TestBar_LineTwoListsSessionsPrimeFirst`
- `internal/reedengine#TestBar_LineTwoShowsTheCheckMarkBeforeTheClock`
- `internal/reedengine#bindingsFixture.click`
- `internal/reedengine#TestBindings_StatusBarClicksSelectAndZoomByRangeType`

**Uses:**
- `internal/reedengine#listPrologueFormat`
- `internal/reedengine#newBarFixture`
- `internal/reedengine#newBindingsFixture`
- `internal/reedengine#installFakeTmux`
- `internal/reedengine#newTestEngine`

**Intent:**
Give the bar a third line, the action row, and move `follow` there as its first entry (D1).

`bar.go`:
- `statusFormatSessions` (line 2) drops `followButtonFormat`: the session loops, then `#[nolist align=right]`, the check mark and the clock, unchanged otherwise.
- `statusFormatActions` is `status-format[2]`: `listPrologueFormat`, the follow button, then `#{?#{@strand},#{E:` + `buttonRowAgentOption` + `},#{E:` + `buttonRowPlainOption` + `}}`, so a window whose active pane carries `@strand` draws the agent row and every other window the plain row.
  Both options are unset until card 6 renders them, so every session shows `follow` alone on line 3 after this card.
  The row is left-aligned in list mode with the `<` and `>` markers, so a row wider than the client scrolls like the other lines.
- `followButtonFormat` keeps its range, look and click branch; only its doc comment changes, from "sits left of the time" to "the first entry of line 3".
- The two row option names are declared beside the bar's other options as display only.
- The file's header comment describes three lines: line 3 is the follow button then the action row read from the two row options.

`windowsize.go`:
- `statusFormatPinArgvs` returns the global pins of every status format and the bar style: `set-option -g status-format[0]`, `[1]`, `[2]` with `statusFormatButtons`, `statusFormatSessions` and `statusFormatActions`, then `status-style` with `statusBarStyle`.
- `pinGeometryOptionsLocked` issues those first, then `status 3` (was `2`) and `status-position bottom` on the session, each non-fatal as before; its doc comment and the file header say three-line bar.
  `status-format[2]` is set with `-g` beside the other two, since a per-session index would hide the global lines.
- `repinStaleBindingsLocked`, when the stamp is stale, issues `statusFormatPinArgvs` before `pinBindingsLocked`, each failure logged at Warn and ignored, so the first reed op of the newest binary restores the global formats an older binary pinned.
  Bound, from D1: between an older binary's boot or attach and the newest binary's next op, `follow` can show on lines 2 and 3; a session an older binary boots or attaches keeps `status 2`, without line 3, until the newest binary next attaches it, since the re-pin restores only the global options.

`doc.go` (reedengine) changes every sentence that says two-line bar, that puts the follow button on line 2 or left of the time, and that says "the global status-bar options refresh only at boot and attach", which now reads that they also refresh with the bindings' stale-build re-pin; it states the Bound above.

Tests:
- `TestPinGeometryOptionsLocked` (`tier0`): `wantOptions` gains `set-option -g status-format[2] statusFormatActions` after `[1]` and reads `status 3`; the two counts of every attempted pin, 8 and 9 in `OneOptionFailureDoesNotStopTheRest` and 8 in `FailedUnpinsAreNonFatal`, each grow by one.
  Its doc comment and its `//testtiming:keep` reason, which say the two global status formats and status 2, name the three formats and status 3.
- `TestBindingsBuildStamp`'s `OpLockRepinsStaleBindings` (`tier0`) asserts, on each row that re-pins, that the `set-option` calls issued before the first `bind-key` include every argv of `statusFormatPinArgvs()` in order, so dropping `status-format[0]`, `[1]`, `[2]` or `status-style` from the re-pin fails; on the other rows none of them is issued.
  A new subtest `FailedFormatRepinIsNonFatal` beside it, whose fake answers a stale stamp and fails every `status-format` `set-option`, asserts the bindings are still re-pinned and one Warn is logged per failed pin; it reads the logs through `logcapture.CaptureVerbose`, so it does not call `t.Parallel`, and a comment names the logger's process-global output as the reason.
- `TestAttachArgv_PreflightOnAKnownGoodSession` (`tier0`): `wantSetOptionCalls` is `13 + wantShellPins`, and its comment names six bar pins.
- `TestStatusFormats_SessionLoopsTakeOneFormat` (`tier0`) scans `statusFormatActions` too.
- `TestBar_LineTwoListsSessionsPrimeFirst` (`tier2 && linux`): its helper `assertSessionButtonsOnce` asserts line 2 holds no `range=user|follow`, and its no-comma check, which sliced the line up to the follow button, takes `nolist align=right` as its anchor instead, so the session buttons sit directly before the right-aligned segment with no comma between; the follow rows expand `#{T:status-format[2]}` and assert the lit and unlit follow button is the first range of line 3.
- `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` (`tier2 && linux`): its running-mark error text, "want the mark right-aligned after the follow button", reads "after the session buttons"; the assertions are unchanged.
- `bindingsFixture.click` takes the row among three status lines, the line `row` sitting `2 - row` lines above the client's last row.
- `TestBindings_StatusBarClicksSelectAndZoomByRangeType` (`tier2 && linux`) clicks `follow` on row 2, and gives its odd session `status 3`.

**ImpactSummary:** the bar's line 2 format and the geometry pin's option list change, and the op lock's stale re-pin issues the global formats; the tests asserting the pin list, its counts, line 2's follow button and the click rows change with it; no production caller outside reedengine reads these formats.

**Commit:** 5: Add a third status line and move the follow button to it
