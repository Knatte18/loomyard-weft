```yaml
lens: interactions
outcome: reported
```

I reviewed the interactions lens: pairs of decisions that act on the same state, and each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 / D2 (the ids D2 reads come from D1's headings; both run beside `Validate` in both callers): held.
- D1 / D4 (H3 headings against the glyph shape; D4 excludes backticked headings): held.
- D2 / D3–D4 (a record missing or unreadable at the gate): broken, see F3.
- D2 / D4 (both skip `Added after Discussion` entries): held.
- D3 / D4 (a token holding `#` or `:` goes to D4 or neither; a `/` path never matches `pkg.Member`; marks shared, so a marked token gets neither finding): held.
- D3 / D4 against the skip list (fenced blocks, suffixed PATTERN bullets, carry-over blocks; stated once for both): held.
- D4 / D8 (D8 classifies through D4's resolving method, and a quarry outage is an error on both sides): held for member glyphs, broken for `plan:` handles, see F1.
- D4 / D9 (`planContextFindings` gains the index; `ValidatePlan` and `ValidateReworkPlan` already hold it and already call quarry first): held.
- D4 / Constraints (`discussionparser` stays stdlib-only, `planglyph` owns quarry calls): partly broken, see F4.
- D5 / D6 (D6 relies on the markers being complete for every struct a test reflects over): broken, see F2.
- D5 / guard scan (`//lym:seam`, `//lym:guard` and `//testtiming:keep` stack in any order, and both parsers admit the others): held.
- D5 / D9 (markers read at check time through the `impactset` parser over the worktree root): held.
- D6 / D8 (same Uses / later-Edit / package-self-glyph cover rules, matching `uses-later-target` and `redundant-package-target`): held.
- D6 / D8 (`plan:` handle targets: D6 strips the prefix, D8 is silent): broken, see F1.
- D7 / D9 (tier tags from `gate.yaml`, skipped with no `gate.yaml`, Go plans only): held.
- D7 / D5 (no shared state): held.
- D9 / D6–D8 (done cards skipped; "later card" is still among the pending cards): held.
- D1–D2 / D10 (darn records get D1 and D2 through the spec the darn stencil already points at): held.
- Acceptance criteria D1–D11 against their decisions: held. D8's member-glyph bullet matches D4's method, D4's outage bullet matches D4, and D9's bullet is a subset of D9.
- The record against its own D2 rule (D1 to D11 each appear as a whole word in a criterion): held.

**Findings**

F1 — MEDIUM, design, decision-record.md D8 (with D6 and D4).
- **Summary:** D8's test/non-test rule does not cover a `plan:` handle target.
- **Issue:** D6 says a handle's unit and member count once the `plan:` prefix is stripped. D8 classifies only file targets, `_test` pseudo-units, member glyphs D4's method resolves, and unresolved Create member glyphs. A handle is none of these. So a later card's Edit of `plan:internal/x#TestFoo`, a test an earlier card creates, falls under "any other target is non-test code". It then draws a false `doc-go-unlisted`. Create targets for new symbols are themselves handles (`plan:&lt;unit&gt;#&lt;member&gt;` -&gt; head), so the "unresolved Create member glyph whose name begins `Test`" clause may never be what a Create card carries. D4's method also defines its input shape to exclude `plan:` tokens.
- **Evidence:** decision-record.md D6 first paragraph and D8 "A target is test code when…"; loom-plan-spec.md:257, 264, 267 and 449 (handles are placeholders for symbols that do not exist yet and appear in card targets and Uses).

F2 — MEDIUM, design, decision-record.md D5 (with D6).
- **Summary:** D5's scan checks that a marker exists, not that every reflected struct is marked, but D6 depends on completeness.
- **Issue:** D5 requires "one line per struct" and says the marker "covers every struct a test reflects over". The scan only refuses a reflecting test that has no marker at all, plus a marker whose glyph is not a struct. Two cases slip through:
  - `envkit.NilSeams` → `walkNil` recurses into every nested struct field of `shedrecipe.Env`. A NilSeams test marked only for `Env` passes the scan.
  - `assertStructFieldsSideFree(t, v any)` reflects over whatever struct it is handed.

  A card adding a func field to a nested struct of `Env`, or to a struct a helper is handed, is then never held by D6. The Bound names only the other-package-helper gap, and the Rationale ("a marker kept complete by a scan is exact") overclaims.
- **Evidence:** internal/testkit/envkit/envkit.go:521–545 (`NilSeams` and `walkNil`, recursion on `reflect.Struct`); internal/fabricengine/mergevocab_test.go:199–205; decision-record.md D5 paragraph 1 and Bound.

F3 — LOW, design, decision-record.md D4 (against D2).
- **Summary:** D2 says what its checks do on a record `Validate` reports missing; D3/D4 say nothing.
- **Issue:** D2 states that D1 and D2 "add nothing on a record `Validate` reports missing". D3 and D4 run in the same `ValidateLoomDiscussion` and go through `discussionparser` extraction, which reads the record. On a missing record that read is a returned error. A missing-record re-prompt would become a gate error, which halts the gate. The record needs the same "nothing on a missing record" rule for D3/D4.
- **Evidence:** decision-record.md D2 paragraph 2 and D4 paragraph 5; internal/loomshed/gates.go:123–133 (`ValidateLoomDiscussion` returns read errors as errors).

F4 — LOW, consistency, decision-record.md D4 (against Constraints).
- **Summary:** D4 has `discussionparser` "extract the citations", but deciding what counts as a glyph citation needs quarry's grammar, which a stdlib-only package cannot import.
- **Issue:** A glyph citation is defined as a token that "parses under quarry's glyph grammar" (quarry `glyph.Parse`). The constraint keeps `internal/discussionparser` stdlib-only. Today the non-`planglyph` importers of quarry are `planparser`, `batcher`, `failuretrace` and `quarrycli`. The record should say that `discussionparser` extracts raw backticked tokens and that `loomshed` applies the glyph and `pkg.Member` shape rules.
- **Evidence:** decision-record.md D4 first bullet and paragraph 5; Constraints bullet 5; quarry v0.4.0 glyph/parse.go:14; the grep of quarry importers.

**Tool calls**
1. Grepped `internal/planglyph/doc.go` and the production importers of quarry, to test whether D4's glyph parse fits the quarry-ownership and stdlib-only constraints. My context lacked the importer list.
2. Grepped loom-plan-spec.md for `plan:`, to confirm handles appear as card targets and Create spellings. My context lacked the plan format's handle rules.
