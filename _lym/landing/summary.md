# Type the start prompt when a long strand name fills the box's top rule

A spawned Claude session whose strand name nearly fills the input box's top border never got its start prompt.
In a narrow pane Claude draws that border as `─ <strand name> ─`, with one rule glyph on each side of the label.
`isTopBoxRule` in `internal/shuttleengine/claudeengine/session.go` took a labelled top rule only when it opened with three rule glyphs or ended in a cut label (`… ─`).
It missed this shape, so the idle check read "no input box is on screen".
Every send of the start prompt was held busy, the retries hit the same check, and the start ended with the prompt untyped (issues #195 and #196).

## Changes

- `isTopBoxRule` now takes any line that starts and ends with a rule glyph as a labelled top rule, whatever the label says.
  The cut-label form and the plain rule still count.
  A label with no rule glyph after it still does not, and the bottom rule must still be a plain rule,
  so a draft line shaped like a labelled rule below the caret still never closes the box.
  `TestInputBoxText` gains a row built from the pane capture in #196, which reads no box on the old check.
- `sendStartPrompt` in `internal/shuttleengine/run.go` now tells the parent, through the runner's notifier, when the start prompt is still held busy after its last send.
  The notice names the strand and the last hold's reason, delimited as agent text, since the reason may quote the input box.
  Before this, such a start showed only as a warning in the log.
  Only an autonomous run on a runner with a notifier sends the notice, and a notifier error is only logged.
  `TestStartup_SendFailuresTearDown` now asserts the notice on every row whose prompt stays held busy, and no notice on the others.

## Design decisions

- A respawn on the same row no longer repeats the failure because the box check itself is fixed.
  The respawn path is unchanged.
- The notice fires when the retries run out or the run's deadline cuts them short.
  It does not fire for a held skill-load send, which has its own failure path.
- Whether this hub's own sighting on 2026-11-23 had the same cause cannot be settled from the code.
  The #196 capture shows this cause, and the fix covers any strand name of that shape.

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
