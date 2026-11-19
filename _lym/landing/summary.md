# Deliver the review gate's repair prompt past a cut pane label

Issues #151 and #152 report runs that halted `failed` at `Code-Burler-Review` with `producer-hard-failure`,
because the reviewer's review file had frontmatter that did not parse as YAML (`mapping values are not allowed in this context`).
A third run, `creel-slack-post-silent-when-off`, halted the same way.

## Root cause

A repair path already exists.
The review-parse gate entry re-prompts the reviewer up to three times with the parse error.
In all three halted runs the trace shows the gate failing and the re-prompt never sent:
`shuttle: gate: re-prompt held by a pane with no input box past the bound, ending the loop with the attempts spent so far`.

The reviewer's pane did show an empty input box.
Its top border carried the session label, which was too long for the pane, so Claude Code cut it with `…` and drew no rule glyphs before it:

```
 ly:creel-slack-post-silent-when-off:code-burler… ─
❯ 
───────────────────────────────────────────────────
```

The Claude engine's `isTopBoxRule` accepted a labelled top rule only when it starts with `───`, so it found no box.
Readiness then held every re-prompt as "no input box" until the bound expired.
The review entry is pass-on-cap, so the run went on with an unrepaired file, and the strict parse in `acceptReview` failed the round.

## What changes

- `internal/shuttleengine/claudeengine`: a line ending in a cut label and one rule glyph (`… ─`) counts as the input box's top rule.
  The gate's re-prompt now reaches a reviewer in such a pane, so an invalid review file gets its repair turns before the round fails.
  The fix is in the engine's box reader, so every send that reads the box gains it, not only the review gate.
- `contracts/stencils/burler/burler-review-format.md`: the review format now states the rule for every frontmatter value, not only `summary`.
  A value holding `: ` or opening with a YAML indicator character is one double-quoted string with its `"` and `\` escaped.
- `internal/burlerengine`: a frontmatter parse error caused by an unquoted `: ` now carries a hint saying to quote the whole value.
  The re-prompt and the halt reason both show it.
  The existing hint for a quoted summary fragment is unchanged.
- `internal/haltnote` and `internal/loomcli`: a halt note's history row for a step that handed the run on now ends with `routed to <producer>`.
  `Code-Bouncer / stuck` before a `Code-Burler-Review` halt reads as `- Code-Bouncer / stuck / <at> / routed to Code-Burler-Review`,
  so the note shows that the Stuck was a routine hand-off and the halted producer is the one whose output was invalid.
  `Parse` reads the optional fourth field and still reads older three-field rows.

## Design decisions

- The engine's top-rule check is the fix, not a fallback in the shed producer.
  A fresh second reviewer attempt on an invalid file would repeat a whole review to fix a formatting slip, and the repair path the issues ask for already exists.
- The issue's suggestion of a verb that writes the frontmatter for the reviewer is left out.
  With the re-prompt delivered, the gate's repair turns and the stated quoting rule cover the slip; such a verb would change the reviewer's whole output contract.
- The cut-label check matches only the ellipsis followed by a space and one rule glyph.
  A label with no rule glyph after it still reads as no top rule, as before.

## Tests

- `TestInputBoxText` gains the halted pane's capture as a row that reads an empty box,
  and a row where a cut label with no rule glyph after it is no top rule.
- `TestParseReview` gains a row where an unquoted `: ` in a summary yields the quoting hint.
- `TestRenderParse` gains a round-trip row for a routed Stuck row,
  and `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory` checks that the loom halt note carries a history entry's route.
