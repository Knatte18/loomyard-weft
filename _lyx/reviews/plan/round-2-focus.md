---
round: 2
focus:
  - "In the plan's card for the burler halves engine, check that both the review table and the fix table carry a segment color on the launch spec (`seatengine.Table.Segment` has no default), and that a test pins it, per `PATTERN-spawn-color`."
  - "For each Edit card that deletes a named existing test, fake or helper in a test file, check whether the card lists those members as Delete glyphs instead of a bare test-file path, without tripping `redundant-file-target`."
---

Both entries target the places where the plan's text and its listed targets could disagree with its Intent.
