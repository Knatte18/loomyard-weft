# Find the input box under a long strand label; notice a held start prompt

A spawned session could sit at an empty prompt, its task prompt never typed, with no notice to the parent.
shuttle's idle check did not see the input box when the strand name nearly or fully filled the box's top border,
so it held the start prompt as busy on every send and its retries.
`isTopBoxRule` in `internal/shuttleengine/claudeengine/session.go` took a labelled top rule only when it started with three rule glyphs or ended in `… ─`.
A label with one `─` on each side (`─ dop:models-torque-drag-adapter:plan-burler-review ─`) matched neither.
A label that filled the pane with no ellipsis (` ly:run:code-burler-review ─`) matched neither.

Closes #195
Closes #196

## What changes

- `isTopBoxRule` takes a line that starts and ends with a rule glyph, whatever its label says.
  Before, it needed three leading glyphs.
- It takes a line ending in a space and one rule glyph (`labelledRuleEnd`), with or without an ellipsis before it.
  Before, it needed the `…`.
- The bottom rule still has to be a plain rule, so a draft line shaped like a labelled rule below the caret never closes the box.
- A start prompt that is still held busy after its last send now notices the parent once, naming the strand.
  `Run.sendStartPrompt` calls the new `noticeStartPromptHeld` on the last hold,
  whether the retry cap or the run's deadline ended the retries.
  The start still ends as before: prompt untyped, strand torn down, the error carrying the pane's last lines.

## Design decisions

- The notice follows the existing held-turn notices.
  It goes only to an autonomous run (`Spec.Interactive` false) on a runner with a notifier, and a notifier error is logged and never ends the run.
- The hold's reason can quote the input box, so the notice delimits it as agent text like the other notices.
- The notice names the strand by `StrandName` and falls back to `StrandGUID`.
- The input-box fix is a loosening of the top-rule match, not a search for the `❯` line with borders around it.
  The top-rule check still decides where the box starts.
- I did not judge whether the 529 Overloaded errors from the same morning contributed to the sighting in this hub.

## Tests

- `TestInputBoxText` gains a row for the one-glyph-each-side label from the #196 capture and one for a label that fills the pane with one glyph after it.
  The existing row for a cut label with no glyph after it stays a non-match.
- `TestStartup_SendFailuresTearDown` now installs a notifier and asserts per row:
  exactly one notice naming the held start for each held-busy row, carrying its delimited reason,
  and none for an interactive run or for a start that is not held busy.

## To check by hand

- Top rule matching, `isTopBoxRule` in `internal/shuttleengine/claudeengine/session.go`:
  a label ending in a space and one rule glyph counts as a top rule with no glyph or ellipsis before the label.
  Check that no line a draft or agent output can produce above the caret matches by accident and opens a false box.
- Nothing was run against a live session with a narrow pane and a long strand name.

## Merge danger

**Door:** two-way
**Blast radius:** shuttleengine
