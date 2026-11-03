# Round 1 fixer-report

## Changes

- F1 (fixed differently): `rootRelativeGlyphRemedy` in `internal/planparser/validate.go` now reads "; root: is never joined to a glyph or a //-escaped path, so this names the worktree-root file %q; to name the file root: joins instead, spell it as %q, or drop the // escape or the trailing \"#\" so root: joins the path %q".
  The proposed wording said the worktree-root file was the one to "name" with the glyph; the glyph names the root-joined file, so the clause is reworded to say that.
  The proposed `//guide.md` row was not added: `Validate` on a hand-built `Plan` skips the canonicalization that turns `//guide.md` into `guide.md#`, so no row can reach that spelling.
  The existing `JoinedPathExists` row asserts the new wording.
  Commit `674706ec`.
- F2 (fixed as proposed): `contracts/specs/loom-plan-checks-spec.md` line 64 now says "present on disk or created by a card".
  Commit `a8d43742`.
- F3 (fixed as proposed): the `self glyph spelled relative to root:` subtest in `internal/planparser/validate_test.go` gained a `ref` column and the row `PathShapedRefGetsNoRemedy` (Edit card, ref `sub/dir`, `docs/sub/dir` present, no remedy wanted).
  `glyph.Parse` returns `ReasonNoSeparator` for a ref with no `#` (quarry `glyph/parse.go:27-30`), so the `ParseRef` guard fires on this row and removing it makes `glyph.Self` produce a remedy that the row rejects.
  The subtest's remedy marker is now `spell it as "`.
  Commit `674706ec`.

Gate: `lym gate test ./internal/planparser` passed (tier0, 886 targets, 0 failed).

## Self-check

- fix-consistency: no report findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the F1 `//guide.md` row was not added — fixed differently, named under F1 above.
- fix-proposal, LOW: the F3 row may not exercise the guard — premise false: `glyph.Parse` rejects a `#`-less ref (quarry `glyph/parse.go:27-30`), so the guard fires.
- fix-proposal, LOW: the `//` clause is noise for a plain glyph writer — disputed on `every fix worse`: the remedy cannot tell the two spellings apart, so a shorter text would blame one of them.
- fix-code-correctness, LOW: the F3 row may not fail without the guard — premise false, same evidence as above.

## Disputed

- fix-proposal LOW (the `//` clause is noise): ground `every fix worse`. Evidence: `rootRelativeGlyphRemedy` gets a canonicalized `guide.md#` for both a written glyph and a `//` escape, so a wording naming only one spelling is wrong for the other.

## Deferred

None.
