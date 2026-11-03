# path-missing names the fix for a self glyph spelled relative to root:

A plan whose overview sets `root:` can carry a card ref such as `README.md#`, a file self glyph the writer meant relative to the module directory.
Glyphs are never `root:`-joined, so that ref names a file at the worktree root.
`lym webster run` then refuses the plan with `path-missing` and `glyph-not-found`.
The plain path `README.md` under the same `root:` is joined and canonicalized to `<root>/README.md#`, so it passes.
The finding said only that the path does not exist, so neither the plan writer nor a fixer could tell which spelling to use.

## What changes

- `internal/planparser`'s `path-missing` check: when the reported ref is a self glyph, the plan sets a `root:` other than `.`, and the `root:`-joined path is present (on disk, or as a Create or Rename destination), the finding's detail now names the worktree-relative self glyph and the plain path `root:` would join.
  Example: under `root: docs`, `guide.md#` gets "spell it worktree-relative as `docs/guide.md#`, or drop the trailing `#` so root: joins the path `guide.md`".
  The check's verdict is unchanged: only its detail text gains the way forward.
- `contracts/specs/loom-plan-spec.md` now states that a file self glyph is worktree-relative under any `root:`, with the `README.md#` against `README.md` example.
- `contracts/specs/loom-plan-checks-spec.md` documents the new way forward on `path-missing`.

## Design decisions

- **One parser already serves both sides.**
  The entry asks for one path-with-anchor parser shared by the plan check and Webster.
  That sharing already exists: the plan gate (`loomshed.ValidatePlan` through `index.ValidateFormat`) and Webster's run and begin-batch validation (`index.ValidateDispatch`) both run `planparser.ValidateFormat` and `planglyph`'s `resolvePass`, against the same worktree root (`location.WorktreePath()` in both the shed env and `hubgeom.WebsterGeometry`).
  A scratch plan run through `lym webster validate` reproduced the reported pair of findings on a `root:`-relative `lymcwd.md#`, and code reading shows the plan gate reports the same pair.
  The halt notes from the original run were not available, so how that plan got past review could not be traced; this change makes the finding say how to fix the ref, wherever it is first reported.
- **No change to how glyphs resolve.**
  The entry's other option, root:-joining an empty-anchor glyph, would contradict the plan spec's rule that glyphs are never `root:`-joined.
  It would also silently re-point every existing worktree-relative file self glyph under a `root:`, and the spec already prescribes one for an extensionless file under `root:`.
  The finding names the fix instead.
- The remedy is offered only when the joined path is present, so it never points at a second missing path.

## Tests

`TestValidate_PathMissing` gains a table subtest: under `root: docs`, `guide.md#` reports one `path-missing` whose detail names `docs/guide.md#` when `docs/guide.md` exists, and leaves the remedy out when it does not.
