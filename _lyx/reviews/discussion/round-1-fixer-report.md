# Fixer report — discussion, round 1

Every finding's premise held against the code: `resolvePass` calls `CanonicalizeHandles`, which writes through `planparser.RewriteRefs` and is followed by a `ParsePlan` reload; `validateFormatAfter` runs `ValidateDispatch` and then `planGatePass`; `laterDeleteReferenceReasons` feeds `recoverbatch.go`'s `ErrRecoveryDeleteReferenced`; `candidateFiles` skips `testdata`, `vendor` and `.`/`_` directories; `planparser.AmendmentsFileName` is `amendments.md`.

## Changes

- F1: Decision 5 now requires handle canonicalization to write nothing in overlay mode, substituting in the overlay reader's contents in memory, with the re-parse reading through the same reader.
  Decision 6 makes the reviewer's baseline `validate-plan --overlay` with `{}`, never the flag-less run.
  Acceptance row added: a non-canonical plan handle overlay leaves every plan file byte-identical and later checks see the overlaid card.
  Of the two options the `fix` key offered, the in-memory substitution was picked, since the acceptance row requires later checks to see the overlaid card.
  A plan-writer note names the write and reload sites.
- F2: Decision 1 names the entry points running the typed form and drops `delete-before-reference` from the resolve-backed pass at the plan gates; acceptance row added for a plan gate with a done card.
- F3: Decision 4 and Scope's `internal/websterengine` bullet state `recover-batch`'s refusal fires on blocking findings only; the Webster acceptance row is extended.
- F4: Decision 3's bound now states the skipped go-tool-ignored directories and why they never break the build.
- F5: Decision 5 narrows accepted names to `00-overview.md` and the on-disk Card Index's card files; the refusal row lists `amendments.md` and an orphan `.md` file.
- F6: terminal stdin added to the refusal row; a row added for the package self glyph Delete's `.md` versus `.go` import-path match.
- F7: Decision 6 states the solo-run procedure and the combined-run fallback, and restates the bound as one baseline, one combined, and one solo run per proposed fix.
- F8: Decision 7 names the file's self glyph when no member span holds the cited line.
- Support log: round 1 recorded under Review rounds, and three new auto-picks added to the Question ledger.

## Disputed

None.

## Deferred

None.
