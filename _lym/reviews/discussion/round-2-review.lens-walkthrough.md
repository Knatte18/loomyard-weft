```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision on one concrete case and checked whether the end state meets the decision's stated purpose.

**Decisions walked**
- D1, a record with `### D1 — x` and an `### Added after Discussion` entry: purpose met.
- D2, D1 named only inside "D10", and the range "D1 to D4": purpose met. A whole-word match keeps D1 out of D10, and the range counts only the ids it spells.
- D2 darn caller, a record `Validate` finds defective: purpose met. D1 and D2 are appended, and the history check is skipped.
- D3, `internal/loomshed/plancontext.go`, `_lym/discussion/x.md` and `internal/x.go:42`: purpose met. The first is tested; the `_lym` root and the `:` token are untested.
- D4, `loomshed.ValidateLoomDiscussion` resolved as `internal/loomshed#ValidateLoomDiscussion` through the new `Index` method: purpose met for found, not-found, a partitioned ambiguous answer and an outage. Two cases are not met: an unpartitioned ambiguous answer or rejected answer, and a non-Go hub (findings 1 and 2).
- D5, `TestConfigTemplate` (`typ.NumField`), the `assertStructFieldsSideFree` helper and `TestNilSeams_ReportsExactPaths` (unqualified `NilSeams` in package `envkit`): purpose met. Each is caught by the stated trigger.
- D6, a card with Edit file target `internal/shedrecipe/recipe.go` against tests marked `internal/shedrecipe#Env`: purpose not met as specified. No step supplies the struct's declaring file (finding 3).
- D7, a card that creates `internal/foo/bar_test.go` with an Intent line "`bar_test.go`: `//go:build tier0`": purpose met.
- D8, Edit `internal/loomshed#TestNewDiscussionGate`: purpose met. The method reports `gates_test.go` only, so the target counts as test code.
- D8, Create `internal/loomshed#fakeGate`, a test helper not yet declared: purpose not met (finding 4).
- D8, an Edit `plan:` handle of a test an earlier card creates: purpose not met (finding 4).
- D9, `ValidatePlan` → `planContextFindings(plan, pending, …)` with the index added: purpose met.
- D10 and D11 are text placement only, with no mechanism to run: purpose met.

**Findings**

1. MEDIUM, design. Location: `decision-record.md` D4, the resolution outcomes.
   - **Summary:** D4 hands the new method's per-glyph answer to `loomshed`, but it names only two answers. An unpartitioned ambiguous answer (one member declared twice in a unit) and a rejected answer map to no finding and no error.
   - **Detail:** D4 says the `glyph-not-found` status policy is "lifted out of its per-card form". In that policy, `StatusAmbiguous` (unpartitioned) gives `glyph-ambiguous`, and an absent or unknown status gives `glyph-rejected`. D4 names only `citation-symbol-ambiguous`, which covers the package base name, and `citation-symbol-missing`. Neither a fail-open nor a fail-closed result is stated for the other two answers.
   - **Evidence:** `internal/planglyph/resolve.go:87-148`, the switch arms for `StatusAmbiguous` and the default `glyph-rejected` arm.

2. MEDIUM, design. Location: `decision-record.md` D4, the glyph grammar and the hub language.
   - **Summary:** Step 1 of D4, "parses under quarry's glyph grammar", needs a language, and a decision record carries none.
   - **Detail:** `glyph.Parse(lang, s)` rejects any `lang` other than Go or C#. planglyph takes the language from `plan.Language`, and `GlyphLanguage()` gates it. D9 limits the plan checks to `language: go`, but D4 sets no such guard. On a C# or `none` hub it is unstated which grammar parses a glyph and whether D4 runs at all.
   - **Evidence:** quarry v0.4.0 `glyph/parse.go:14-17`; `internal/planglyph/handle.go:228` and `planglyph.go:314`.

3. LOW, design. Location: `decision-record.md` D6 and D9.
   - **Summary:** D6 holds a card to a marked struct when a target names "the file that declares the struct", and nothing supplies that file.
   - **Detail:** The marker gives only `&lt;unit&gt;#&lt;Type&gt;`. D9 hands `planContextFindings` the index "for D8's classification" only. D4's method reports declaring files, but D6 is never said to resolve marked struct glyphs through it. A card whose only target is `internal/shedrecipe/recipe.go` cannot be matched to `internal/shedrecipe#Env`.
   - **Evidence:** `decision-record.md` D6, D9, and D4's "which D8 reads".

4. LOW, design. Location: `decision-record.md` D8, the test classification.
   - **Summary:** Two kinds of target are classified as non-test code, so a test-only card gets a false `doc-go-unlisted`. The Bound says a test-only card is not checked.
   - **Case 1:** A Create member glyph for a test helper whose name lacks a `Test`, `Benchmark`, `Fuzz` or `Example` prefix, such as `internal/loomshed#fakeGate`.
   - **Case 2:** An Edit or Extends `plan:` handle of a test function an earlier card creates. It does not resolve at Plan time and is not a Create target.
   - **Evidence:** `decision-record.md` D8, the classification sentence and the Bound's "a test-only card … not checked".

**Tool calls**
- grep in `internal/planparser` and `internal/planglyph` for `GoPackageCounterpart` and the glyph language use. My context had the `glyph.Parse` signature but not how planglyph picks a language.
- grep in quarry v0.4.0 `internal/engine/answer.go` for `ResolveResult` fields. I needed to confirm a found answer carries `Symbols` with a file, which D8's file report relies on.
