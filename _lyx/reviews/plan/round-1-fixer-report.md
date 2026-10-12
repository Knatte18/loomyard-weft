# Fixer report — plan, round 1

`lyx loom validate-plan` passes after the edits, with only informational new-unit findings.

## Changes

- **F1** (fixed as proposed): card 8 creates `internal/pausecli/deps_test.go`, an untagged file holding `TestShuttlePairReadings`, with the file comment the fix names; the Intent no longer puts the test in the integration-tagged `cli_test.go`.
- **F2** (fixed by closing both paths): card 5's `Runner.Inject` refuses a `Key` naming the interrupt sequence case-insensitively, with tmux's `C-[` read as `Escape`, and refuses any `Text` holding a control character, with its own message naming `lyx shuttle interrupt`.
  The Bound bullet now names what still passes (other tmux spellings of the key) and what bounds it (Inject has no production caller).
  `TestRunner_Inject_RefusesAnyInterruptKey` gains rows `{Key: "C-["}`, `{Key: "escape"}` and `{Text: "\x1b"}`.
  The D7 line in the Intent and the Inject bullet of `pattern/PATTERN-guarded-interrupt.md` match.
  Fixed differently in one detail: the review's bound cites creel reaching its Esc through the guarded `Interrupt`, but no creel package in the tree calls `Interrupt`, so the bound names only Inject's lack of a production caller.
- **F3** (fixed as proposed): card 8's **Edit:** list names `internal/pauseengine#maxInterrupts` after `Pair`.
- **F4** (fixed as proposed): card 8's `fakePair` gains `backgroundErr` and `paneErr`, and `TestPause` gains the row "a failed background or pane reading is refused with its prefix", asserting both reason prefixes.
- **F5** (fixed by the fail-closed option): card 4's `awaitIdleSession` fails closed on a failed picker capture in the branch without `SessionCycler` and keeps the skip only in the branch whose polls hold.
  A new test fake `pickerOnlyEngine` (embeds `fakeEngine`, reads the picker, not a `SessionCycler`) reaches the fail-closed branch.
  `TestGuardedPickerClose` gains two rows: the skip branch (`captureErrs [boom, nil]` on `pickerEngine`) and the fail-closed branch (`captureErrs [nil, boom]` on `pickerOnlyEngine`), each asserting zero `SendKey` calls and nothing typed.
- **F6** (fixed as proposed): `internal/shuttlecli#shuttleCLI.interruptCmd` moves from card 7's **Edit:** list to card 4's, with the `Long` and `Short` help bullets; card 7 keeps its CLI test and budget row and lists `interruptCmd` under **Uses:**.
  Card 7's commit line and ImpactSummary, and both cards' lines in `00-overview.md`'s Card Index, match.

## Disputed

None.

## Deferred

None.
