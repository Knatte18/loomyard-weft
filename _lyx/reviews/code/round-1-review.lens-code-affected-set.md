```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found:**

1. **New per-card-id finding in `CheckCardFabricReference` (`internal/planparser/fabricref.go`).**
   - Production callers: `internal/planglyph/index.go:50` (`ValidateFormat`) and `:59` (`ValidateRework`). Both pass the findings through `convertAll`, so the new finding reaches them unchanged.
   - Consumer tests: the only test I found that asserts the check's findings is `fabricref_test.go`. `internal/loomcli/parity_test.go:375-382` expects `card-fabric-reference` from a verify-section command and is unaffected.
   - Fixtures: a grep for card ids ending in `-weft` across `internal`, `contracts`, `docs/overview.md`, `PATTERN.md` and `pattern` found none outside `fabricref_test.go`. No existing plan fixture newly fails.
   - Finding order: the id finding goes before the card's own span scan, so overview findings still come first and each card's id finding precedes that card's span findings. Existing rows have no id hit, so their count and order stay the same. Covered.

2. **Docs.**
   - `contracts/specs/loom-plan-spec.md` entry 37 and `internal/planparser/doc.go` both describe the new hit, and both match the code: the card and the matched id are named, and the way forward is to rename the card in the Card Index and its file name.
   - `contracts/stencils/loom/loom-template-plan.md:95` only points at the spec entry. The stencil needs no change, since it restates nothing.
   - `internal/planglyph/doc.go:28` and the `internal/loomcli/validate.go:179` help text say "card-fabric-reference" generically. They need no change. Covered.

3. **`internal/fabricengine/refscanner.go`.** Untouched, as the decision record requires. `RefScanner.Matches` and `MatchPath` behave as before.

4. **Done-card handling (not covered).**
   - **[LOW] [scope] `internal/planglyph/index.go:41-44`, `internal/planparser/fabricref.go:72-76`.**
     - `ValidateFormat` receives the `done` set and uses it to exempt completed cards from `edit-test-file`, because a completed card's file has already landed and no writer can fix it (`internal/planparser/edittestfile.go:19`).
     - The new id finding ignores `done`, so format validation also refuses a plan with an already-landed `-weft` card.
     - The decision record's Open risks accepts this only for rework. The spec and `doc.go` text say nothing about it, though the spec does say the check runs in format validation.
     - The record accepts the behavior, so this is not a defect, and the diff matches the record. If the handler wants the spec to be complete, it could note there that done cards are not exempt.

I found no other findings.

**Tool calls (2 of 6):**
1. Grep for `-weft`-suffixed card ids in fixtures and docs, plus a read of `internal/planglyph/index.go:30-70`. The manifest had the callers' line numbers but not their bodies or any fixture search.
2. Grep of the plan stencils and the spec for `card-fabric-reference` and "weft" mentions. The manifest did not cover stencil wording.
