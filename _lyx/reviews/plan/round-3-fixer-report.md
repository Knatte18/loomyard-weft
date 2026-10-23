# Plan round 3 — fixer report

## Changes

- **F1 (MEDIUM, scope), fixed as proposed.**
  Card 19 (`_lyx/plan/19-arbiter-amendment-target.md`) lists `internal/loomcli/doc.go` in `**Edit:**`, between `internal/loomcli#arbiterFixture.deps` and `contracts/stencils/loom/loom-template-arbiter.md`.
  Its Intent gains the paragraph on the Arbiter section of `doc.go`, placed after the stencil paragraph and before "What it lets through, and the bound".
  The self-check then reworded the paragraph's source of the value to "the run's recipe's `amendment_target` key (loom's `Plan-Burler-Review`; darn and batten set none)".
  The review's wording, "the loom recipe's key", misstates the value for darn and batten runs.
- **F2 (LOW, consistency), fixed as proposed.**
  Card 24 (`_lyx/plan/24-planparser-target-holds.md`) adds `internal/planparser#PathEntryPath` and `internal/planparser#HandleBody` to `**Uses:**`, after `ParseMintedCSharpID`.
  Its Intent gains the file-rule sentence after "or the target is `file` as a path or as its file self glyph."
  The self-check then added a clause that compares a target `PathEntryPath` does not read with `file` as a plain path.
  `PathEntryPath` returns false for a ref with no `#`, so without that clause the "as a path" arm had no reading.

`lyx loom validate-plan` reports `ok: true` after every change, with only informational `create-new-unit` findings on card 25.

## Self-check

- fix-consistency, LOW: card 19's doc paragraph says "the loom recipe's key", but the code reads the run's recipe. Fixed as proposed (duplicate of the fix-proposal MEDIUM below).
- fix-proposal, MEDIUM: the same card 19 wording documents the wrong rule for darn and batten runs. Fixed as proposed.
- fix-proposal, LOW: card 24's sentence routes every file comparison through `PathEntryPath`, which refuses a plain path (`internal/planparser/glyphref.go:42`, quarry `glyph/parse.go` `ReasonNoSeparator`). Fixed as proposed.
- fix-claims: no findings.
- fix-affected-set: no findings. It checked both F1 and F2 and left none unchecked.

## Disputed

None.

## Deferred

None.
