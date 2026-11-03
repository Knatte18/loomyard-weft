---
round: 1
focus:
  - "internal/boardengine/render.go: grep every caller of `slugCell` and every place a table row or list renders a slug (Running, Ready, layers, Independent, Draft, Notes, Done, project files, the project facts line); settle whether any table that lists a discussed entry still renders the slug without the Sketch link, against the Expected line 'in every table that lists it'."
  - "internal/boardengine/render.go `slugColumnCell`: settle whether the link target `DecisionFileName(t.Slug)` resolves relative to every rendered file that carries it, given the Outputs paths Render can be handed, and whether the `cellBreak` join keeps the table row valid when the slug cell has no design link (no body)."
  - "internal/boardengine/doc.go and other docs naming render output: settle whether every statement that the README shows, or never names, the decision record file is updated (the 'No render output names that file' wording, the README section, the `Render` doc comment, and any spec or readme under docs or _lym/specs that describes the row cells), per the repository's rule that docs for a change land in the same diff."
  - "internal/boardengine/render_test.go `TestRenderSlugCellLinksTheDecisionRecord`: settle whether the test pins both renderings the Goal asks for (flagged and unflagged) in the tables that matter, and whether its Done row and project-root.md assertions cover behavior not already pinned by an existing render test; check the row strings against the real column order for Ready and Done."
  - "Other consumers of rendered rows (`lym board list`/`find` output, any golden or fixture file holding a rendered README, any parser of a Slug cell): settle whether the added `<br>` line in the Slug cell breaks a reader that expects the cell to hold the slug alone."
  - "Commit message and `_lym/landing/summary.md` claims against the diff: settle whether the summary's 'every table and board file' and 'project facts line keeps plain slugCell' claims hold, and whether the stated placement choice matches the code."
---

Seed focus for round 1. The diff is three files in `internal/boardengine`; the measuring stick is the amended board entry in the decision record's `## Goal`.
