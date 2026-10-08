# Fixer report — plan, round 1

## Fixed

### F1 [BLOCKING] The color-through-the-send-path decisions have no card

Confirmed against the decision record's two entries added after Discussion and the tree: `Runner.start` (`internal/shuttleengine/run.go:494`) still plays `ColorSequence` raw after `awaitStartup`, and `Runner.TypeColor` (`internal/shuttleengine/session.go:192`) does the same.

Added two cards and their Card Index lines in `00-overview.md`:

- `11-color-through-send.md`: `Engine` gains `ColorCommand(color) string` (the command text, so the color can go through `sendVerified`, which types text); `Claude.ColorCommand` and `shuttlefake.Engine.ColorCommand` implement it.
  `Runner.start`'s color step and `Runner.TypeColor` send it through `sendVerified` after readiness; a failed color send is logged at Warn and never fails the launch.
  The Intent states the attack-surface bound (a dropped color send, bounded by card 4's clear, card 5's teardown and the two send windows).
  Coverage extends the existing tests: `TestRunner_Start_TypesTheSegmentColorAfterStartup` and `TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand` gain rows, `color_test.go`'s and `shuttlefake_test.go`'s assertions are retargeted.
  The tmux-tier tests the decision names are left to Publish's `publish_verify`, as the plan template requires (`## verify:` compiles the tmux tier and never runs it).
- `12-delete-color-sequence.md`: deletes `Engine.ColorSequence`, `Claude.ColorSequence` and `shuttlefake.Engine.ColorSequence` once card 11 has removed their last callers, per the spec's rule that removing a symbol's last caller and deleting it are two cards.

I appended the cards rather than inserting after card 4 as the review suggested: they use only cards 3 and 4's symbols, so the card-order rule holds, and appending avoids renumbering cards 5–10.

### F2 [NIT] Card 10's ImpactSummary pins a tally

Replaced the count with the conclusion: "No remaining caller of any deleted symbol, so the build and every behaviour stay unchanged."

## Disputed

None.

## Deferred

### The gate's findings on cards 1–10, left on purpose by the parent's call

Every remaining gate finding (`create-already-exists`, `glyph-not-found`, `path-missing`, `caller-uncovered`) is on cards 1–10, which Webster has already run.
The gate resolves the plan against HEAD, and HEAD already holds those cards' code.
No finding names card 11 or 12, and none comes from this round's edits.

`ly:orch` decided to leave cards 1–10 unchanged as the record of work already run.
Cards 11 and 12 stay as follow-up cards after the last begun batch, each with its Card Index line.
`ly:orch` takes the run past the plan gate to Webster and rebaselines the new cards there.
