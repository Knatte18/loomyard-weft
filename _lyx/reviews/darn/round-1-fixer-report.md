# Fixer report, round 1

All six findings fixed as proposed; `lyx gate test` on `internal/planglyph`, `internal/planparser` and `internal/websterengine` is green.

## Changes

- F1 (`b97b9c766`): `internal/websterengine/recordbatch_test.go` gains the `GlyphRejectedWithUncheckableNamesTheFreshRestart` subtest, which asserts the fresh-restart steps and no `recover-batch`.
- F3 (`ec08c6096`): `internal/planparser/glyphref_test.go` gains `TestIsDiskCheckedTarget`, covering a slashed path, a bare root filename and a spaced path as true, and a self glyph, a member glyph, the glyph half of a stripped plan handle and a bare symbol as false.
- F6 (`a6056c312`): `internal/planglyph/donecheck_test.go` puts the `delete gone` row on its own line.
- F2, F4, F5 (`0e33389b2`):
  - F2: `internal/planglyph/donecheck_resolve_test.go` gains the `Rename spaced path landed` and `Rename spaced path skipped` rows.
  - F4: `searchedDescription` in `internal/planglyph/donecheck.go` sorts `elsewhere`.
  - F5: `diskLookup` reads `syscall.ENOTDIR` as nothing existing there, with a regression row `Delete spaced path under a file parent gone`.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
