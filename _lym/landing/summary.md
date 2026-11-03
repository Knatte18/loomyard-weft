# planparser: path-missing names the fix for a root:-relative self glyph

A plan under `root: docs` that names `README.md#` fails plan validation with `path-missing`, because a glyph is never `root:`-joined and `README.md#` therefore names a file at the worktree root.
Authors read the finding as a malformed trailing `#`, when the actual mistake is the spelling relative to `root:`.
This change makes the finding say so and states the rule in the specs.

Closes #34

## What changes

- `path-missing` in `internal/planparser/validate.go` appends a way forward when the plan has a `root:` other than `.`, the ref is a self glyph, and the `root:`-joined path is present on disk or created by a card.
  The remedy names the worktree-root file the glyph resolves to, the worktree-relative self glyph that names the `root:`-joined file (`docs/README.md#`), and the plain path form that `root:` joins.
- The remedy is built by the new `rootRelativeGlyphRemedy`.
  It returns nothing when the ref is not a glyph, `root:` is absent or `.`, or the joined path does not exist either, so those findings keep their old wording.
- `loom-plan-spec.md` states that a file self glyph is worktree-relative under any `root:`.
  `loom-plan-checks-spec.md` records the extra remedy under `path-missing`.

## Design decisions

- The parser still does not split `#anchor` suffixes or `root:`-join glyphs.
  Both the bouncer and Webster already reject the ref through the same `path-missing` check; the fix is the diagnostic, not a changed acceptance rule.
- A path-shaped ref never gets the remedy, even when a `root:`-joined file exists, because for a path the `root:` join is already applied and a remedy would mislead.
- The remedy is gated on the joined path being satisfied, so it is only offered when it points at something real.

## To check

- The issue describes the same plan passing Plan-Bouncer review and then failing at `lym webster run`.
  This change does not alter what either accepts; it was not run against the original plan, so confirm by hand that the bouncer now reports `path-missing` with the remedy for a `root:`-relative self glyph.
- Tests added in `TestValidate_PathMissing` cover the remedy being present, absent when the joined path is missing, and absent for a path-shaped ref.
