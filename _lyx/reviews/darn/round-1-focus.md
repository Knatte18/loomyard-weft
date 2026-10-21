---
round: 1
focus:
  - "internal/boardengine/board.go `boardCriticalSection`: does applying the old-to-new pair before stamping dates and checking run locks leave non-rename writes untouched, and does a run-held dependent really take the rewrite without bypassing the run lock for any other change?"
  - "internal/boardengine/rename.go: does the rename keep id, position and every other field including unknown keys, rewrite `depends_on` of every other entry, and refuse an absent entry, a run-held entry, and an empty, over-length or existing target before any mutation?"
  - "internal/boardengine/rename.go whole-slug mention match: are the boundary rules (letter, digit, `-`, `_`) correct at string start and end, and does `mentions` cover body and brief of every entry including the renamed one?"
  - "internal/boardcli/cli.go: is the `rename {slug|id, to}` verb wired like the other addressing verbs, with payload declaration, help and error envelope consistent with them?"
  - "Render after the write: is the old slug's design doc dropped and the new one written, and does a failed rename leave the old doc in place?"
  - "Docs (`boardcli` and `boardengine` doc.go, `ly:board` skill) against CLAUDE.md's docs-land-in-the-same-commit rule, and the summary's claims against the diff, including the Not covered section."
  - "Test economy: do the new rename tests and the CLI step duplicate existing coverage, and is any added behavior left without a test?"
---

Focus names where to look; the base review of the full diff still applies.
