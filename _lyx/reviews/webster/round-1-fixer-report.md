# Fixer report — webster round 1

## Changes

- F1 (fixed as proposed): `planparser.ParseMintedCSharpID` in `internal/planparser/glyphref.go` parses a quarry-minted C# id; both `glyph.Parse(glyph.CSharp, ...)` calls in `internal/planglyph/resign.go` use it.
  `pattern/PATTERN-glyph-conversion-chokepoint.md` names the helper.
- F2 (fixed as proposed): `PATTERN.md` regains `; nothing else calls os.Getwd or git rev-parse` on cwd-resolution and `, never a template fill` on sparse-hub-base.
- F3 (fixed differently): the proposed change, not counting an unresolved member toward `nonPathTarget`, left the mismatch firing, because the `.cs` path target's own listing already sets `csharpEvidence`; the new `TestGlyphChain_CSharp` row failed with it.
  `languageMismatchFindings` instead returns no finding when a non-path, non-handle, non-Create target has no symbol, listing or listings, so that glyph's own `glyph-not-found` stands.
  It reads no `.Status`, since the status-consumer tripwire test forbids a new consumer.
  Row added to `TestGlyphChain_CSharp`; module doc and function doc updated.
- F4 (fixed as proposed): `renameCSharpDecl` refuses a constructor (`isCSharpConstructor`: name equals the owner's bare name) with `rename-old-unresolved`; the refusal wording in `csharphead.go`, `handle.go`, `doc.go` and `contracts/specs/loom-plan-spec.md` says constructor.
  `TestRenameCSharpDecl_ConstructorIsNamedByItsType` added (plain and generic owner).
- F5 (fixed as proposed): `TestRun_CSharpMemberGlyphsOverlapByOwnerPrefix` in `tools/plandag/main_test.go`, with `writePlanInLanguage`.
- F6 (fixed as proposed): doc comment of `TestValidate_LanguageRecognized` names csharp.

## Gate repair

The gate refused `PATTERN.md` as 57 bytes past its headroom line, because F2's restored wording plus the glyph entry's declared change grew the overview.
The overview is back at its original 15703 bytes:
- the glyph entry reads `a ref's alphabet only `planparser``;
- `PATTERN-testkit` drops `, never duplicated`, which `pattern/PATTERN-testkit.md` already states;
- `PATTERN-review-round` drops `, end on a verdict`, which `pattern/PATTERN-review-round.md` now states.

The two trimmed entries are outside the declared change; the gate's byte limit forced them, and no meaning left the pattern files.

## Self-check

- fix-affected-set, LOW, spec text in `contracts/specs/loom-plan-spec.md:271` still said static constructor: fixed as proposed.
- fix-consistency, LOW, `internal/planglyph/doc.go` mismatch bullet lacked the F3 rule: fixed as proposed.
- fix-consistency, LOW, chokepoint background did not name the new helper: fixed as proposed.
- fix-code-correctness, LOW, F3 change would also hide the mismatch for a Create of an extensionless self glyph that resolves not_found: fixed as proposed; Create targets are counted as non-path targets without needing an answer, as before.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
