# Card 7 — strand-name-pane-option

**Create:**
- `internal/reedengine#strandNamePaneOption`

**Edit:**
- `internal/reedengine#Engine.setStrandPaneOptionsLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine#TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`

**Uses:**
- `internal/reedengine#assertStrandOptionsReasserted`

**Intent:**
A `{strand}` button needs the clicking pane's full strand name, while `@strand` holds only the role (D6).
`setStrandPaneOptionsLocked` sets the pane option `strandNamePaneOption` (`@lym_strand_name`) to the strand's full name `s.Name`, right after `@strand`, wherever it sets `@strand`: at launch (`launchStrandLocked`) and in the attach pre-flight's re-assert (`reassertStrandPaneOptionsLocked`), so a strand spawned by an older lym gets it at the next attach.
An empty name unsets the option, so a stale name never outlives a strand's relaunch under no name.
A failure is logged at Warn and never fails the step, as for `@strand`.
The option is decision-bearing: the click verb (card 8) reads it, so `paneoptions.go`'s header, which says no Go decision reads the strand options back, and `doc.go`'s strand-options paragraph name it as the one pane option a Go decision reads.

Tests, `tier0`:
- `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`'s wanted call sequence gains `set-option -p -t %new @lym_strand_name <name>` after the `@strand` call, on every row with a named strand.
  A new row launches a strand with an empty `Name` and expects `set-option -p -u -t %new @lym_strand_name` in that place.
  A new row scripts the `@lym_strand_name` `set-option` to fail, through the test's existing `answerFunc`, and expects the launch to succeed with the rest of the sequence unchanged.
- `TestAttachArgv_PreflightOnAKnownGoodSession`: `wantSetOptionCalls` is `15 + wantShellPins`, the bound pane getting three options.

**ImpactSummary:** one more pane option per strand launch and attach re-assert; only the launch-sequence and attach-count tests change, since `assertStrandOptionsReasserted` matches `@strand` as an exact argument.

**Commit:** 7: Set the full strand name on each strand pane
