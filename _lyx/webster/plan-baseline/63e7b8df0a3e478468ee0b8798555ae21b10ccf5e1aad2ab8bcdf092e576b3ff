# Card 4 — plan-spec-scope-guard-pointer

**Edit:**
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Rewrite the deviation union paragraph under `## Deferred / forward-compat` in `contracts/specs/loom-plan-spec.md`, per decision D3.
The paragraph keeps its two facts: the mismatch between a batch's changes and its cards' targets is informational, never blocking on its own, and `Uses:` is excluded from the comparison because it names what a card reads, not what it changes.
Its description of the comparison, the "a mechanical guard compares the delta's changed files against the batch's own target glyphs' `UnitPath`-mapped files directly" clause and the format-5 restatement around it, is replaced by a pointer to `planglyph.ScopeGuard` as the definition of what counts as inside a batch's targets.
That pointer merges with the following "See `internal/websterengine`'s package documentation for the verification semantics." sentence into one pointer sentence naming both.
The sentence beginning "This union is defined over each card's flat target set" is dropped.
The paragraph is written one sentence per line; the paragraphs after it are unchanged.

**ImpactSummary:** One paragraph of the source plan spec; the board's deployed copy refreshes through the normal deploy, and no test asserts this paragraph's wording.

**Commit:** 4: Plan spec points the deviation union at ScopeGuard
