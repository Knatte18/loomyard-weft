# Card 3 — index-resolve-glyphs

**Create:**
- `plan:internal/planindex#GlyphVerdict` -> `type GlyphVerdict string`
- `plan:internal/planindex#GlyphFound` -> `const GlyphFound GlyphVerdict = "found"`
- `plan:internal/planindex#GlyphMissing` -> `const GlyphMissing GlyphVerdict = "missing"`
- `plan:internal/planindex#GlyphAmbiguous` -> `const GlyphAmbiguous GlyphVerdict = "ambiguous"`
- `plan:internal/planindex#GlyphAnswer` -> `type GlyphAnswer struct`
- `plan:internal/planglyph#index.ResolveGlyphs` -> `func (index) ResolveGlyphs(worktreeRoot string, glyphs []string) ([]planindex.GlyphAnswer, error)`
- `plan:internal/planglyph#resolveVerdict` -> `func resolveVerdict(worktreeRoot string, r quarry.ResolveResult) (planindex.GlyphVerdict, bool, ambiguity)`
- `plan:internal/planglyph#TestResolveGlyphs` -> `func TestResolveGlyphs(t *testing.T)`

**Edit:**
- `internal/planglyph#statusFindings`
- `internal/planglyph#allowedStatusConsumers`

**Extends:**
- `internal/planindex#Index`
- `internal/planglyph/doc.go`

**Uses:**
- `internal/planindex/doc.go`
- `internal/planglyph#classifyAmbiguity`
- `internal/planglyph#Resolve`
- `internal/planglyph#TestStatusFindings`
- `internal/planglyph#TestStatusEnforcement_NoOutOfAllowlistConsumer`

**Intent:**
The resolving half D4 needs, with the files D6 and D8 read: `planindex.Index` gains `ResolveGlyphs(worktreeRoot string, glyphs []string) ([]GlyphAnswer, error)`, an interface method added in this card's Extends of `internal/planindex#Index`, implemented in `internal/planglyph` as the only quarry caller.

`GlyphAnswer` holds `Glyph string` (the asked glyph, verbatim), `Verdict GlyphVerdict`, `Files []string` (the worktree-relative files declaring a resolved glyph, deduplicated, in answer order) and `Candidates []string` (the files of an ambiguous answer's candidates, each with its build constraint as `candidateList` renders it).
The answers come back positionally, one per glyph; an empty glyph list returns nil and opens no repository.

The `glyph-not-found` status policy is lifted out of `statusFindings`'s per-card form into `resolveVerdict`, which both `statusFindings` and `ResolveGlyphs` call, so the two can never disagree about what resolves:
found and multipart are `GlyphFound` with readable true;
ambiguous is `GlyphFound` when `classifyAmbiguity` reports the candidates partitioned by build constraint, else `GlyphAmbiguous`, with the classification returned for the candidate list;
not_found is `GlyphMissing` with readable true;
a pre-resolution rejection or any status outside quarry's vocabulary is `GlyphMissing` with readable false.
`statusFindings` keeps its findings exactly: readable `GlyphMissing` stays `glyph-not-found` with its unit-branching detail, unreadable stays `glyph-rejected`, and `TestStatusFindings` passes unchanged.
`ResolveGlyphs` reads the files from the result's symbols for found and multipart, and from the partitioned candidates for a partitioned ambiguous answer.

`ResolveGlyphs` resolves through the package's `Resolve`, so `(*quarry.Repo).Resolve` keeps its one call site in `resolveTargets`; a quarry failure is returned wrapping `ErrQuarryUnavailable`, never an answer.
`resolveVerdict` reads the status vocabulary, so `allowedStatusConsumers` gains its row (`internal/planglyph/resolve.go:resolveVerdict`, "fail-closed status reader: an unreadable answer is missing"), and the `statusFindings` row stays only while that function still reads a status selector; `TestStatusEnforcement_NoOutOfAllowlistConsumer` refuses a stale row, so the card leaves the rows matching the code.
The fakes that embed `planindex.Index` (`envkit`'s `noFindingsIndex`, `indexkit.ValidateStub`, shedrecipe's and websterengine's test fakes) still compile, since an embedded nil interface supplies the method, but calling it panics; card 4 gives `envkit`'s `noFindingsIndex` a `ResolveGlyphs` and card 8 gives shedrecipe's `noFindingsPlanIndex` one, where a gate they back first reaches it.

`TestResolveGlyphs`, tier0 in the existing test file of the index, over a temp worktree holding a `go.mod` and a package declaring a struct `T` in `t.go`, a function `F` declared once in `f_linux.go` and once in `f_windows.go`, and a function `G` declared twice under the same constraint: `p#T` is found with `Files` `[p/t.go]`; `p#F` is found with both files; `p#G` is ambiguous with both candidates; `p#Missing` and `nosuch#X` are missing; `p#bad#x` (rejected) is missing; an empty list returns nil; a worktree root that does not exist returns an error wrapping `ErrQuarryUnavailable` and no answers; an answer list is as long as the glyph list and in its order.

Docs: `internal/planglyph/doc.go` names `ResolveGlyphs` beside the status policy bullet, says the policy is shared through `resolveVerdict`, and that it backs the discussion citation check and the plan seam-test and `doc.go` checks.

**ImpactSummary:** one interface method added to `planindex.Index` with its one real implementation; `statusFindings` is refactored onto a shared verdict function without changing any finding, and every fake embeds the interface so none breaks.

**Commit:** `3: Resolve a glyph list through the plan index`
