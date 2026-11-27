# Study every raddle run and tune the split it proposes

`raddle-split-quality` · Task · research, stencil, board · running · Drift-Verify

## Seen (operator, 2026-11-27)

The split a raddle run proposes for a Sketch is too often absurd, and the operator has to revise it before approving.
This is also why arras, which lets a review approve the split, is not trusted.
The operator believes raddle can be tuned so its proposals hold.

## Work

- Find every raddle run made in this hub. Their records are in the weft repository's `archive/<slug>` tags
  (`git -C <weft> tag -l 'archive/*'`, then read the tagged trees' `_lym` records); the board's done history and Bolt's git log name which tasks had recipe `raddle`.
- For each run, put side by side: the Sketch's decision record, the split Raddle-Write first proposed (parts, cut lines, order, dependencies, recipes, profiles),
  every revision the operator asked for in the session, and the split finally filed.
- Class each revision: wrong cut lines, too many or too few parts, parts that overlap a package, wrong recipe or profile, wrong order or dependencies, missing or invented parts, wording, other.
- Count the classes, and quote the operator's own words for each.
- Read the Raddle-Write stencil and recipe, and name for each class the passage that led to it or the rule that is missing.

## Output

A report under `docs/research/raddle/` with the table of runs, the classes and counts, and for each class a concrete stencil change, written as the new text.
Make no stencil or code change in this task; the stencil change becomes its own task from the report.
