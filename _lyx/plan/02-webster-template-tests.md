# Card 2 — Prune webster's template test to Go-filled, Go-chosen and Go-named text

**Edit:**
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#requireNotContains`
- `internal/websterengine_test#TestBeginBatch_KickstartPack`

**Delete:**
- `internal/websterengine_test#prePackStepOne`
- `internal/websterengine_test#prePackFreshRead`
- `internal/websterengine_test#multipartPackFile`
- `internal/websterengine_test#inlineFreshReadBody`
- `internal/websterengine_test#inlineLocateBody`
- `internal/websterengine_test#inlinePackSectionHead`

**Uses:**
- `internal/websterengine_test#promptCheck`
- `internal/websterengine_test#writePrePackBody`
- `internal/websterengine_test#dropInlineMarker`
- `internal/websterengine_test#renderForkWithPack`
- `internal/websterengine_test#renderForkPromptAndPack`

**Intent:**
Prune `TestRenderPrompts` in `internal/websterengine/template_test.go` by D1: an assertion stays when it checks a value Go filled, a fragment Go chose by its inputs (anchored on the shortest phrase that tells the branches apart, or a heading of that fragment), a marker set or a leftover `{{`, or a Go-filled marker's position relative to a heading; an assertion on static prose of one stencil goes.
Row by row:

- "fork carries the card gates and no leftover marker": drop the contains needle "the plan's framing and the repo's rules, and no code" and the notContains needle "this card's package's unit tests"; "recovery carries the card gates and no leftover marker": drop the same notContains needle.
- "fork self-fix section counts card-caused failures" becomes "fork self-fix section carries the self-fix cap": it keeps the cut at the `## Bounded self-fix, then stop` heading and asserts only that the section holds "`7`", the Go-filled `self_fix_cap`.
- "recovery points a cold strand at the overview with PATTERN inactive": keep the card's `SourcePath` and the notContains needles "{{" and "## Constraints"; drop "00-overview.md", "## Your final action: the minimal batch-report", the two prose phrases, and the notContains needles "CONSTRAINTS.md" and "Orient to the codebase".
- "recovery carries the PATTERN overview when PATTERN is active": keep "some constraints" and "## Constraints"; drop "the constraints section above points at, when that section is present".
- "fork with a nil pack …": keep every byte-equality check (file, inline and per-card layouts, and the render of the body `writePrePackBody` builds from the plain fragments, which is what tells the pack-off branch apart); drop the contains needles, `prePackStepOne` and `prePackFreshRead` among them, and the notContains needle "The kick-start pack file lists"; keep the notContains needles "## Kick-start pack", "## Start here" and `testPackPath`.
- "fork with a resolved multipart pack …": keep the "lyx loom card-pack" absence check and the file-layout equality with the `dropInlineMarker` render; drop the `multipartPackFile` comparison, and the row's inline comment stops saying the pack file equals a literal copied from a render before the layout switch: it says the file layout equals the render of a body without the inline marker.
  Its contains list becomes the specs directory alone (`specsDir`); its notContains list keeps "Locate them with one", "{{", "no member glyphs on this card" and "#### Unresolved" and drops the rest; its inOrder list keeps "## The FRESH-READ rule", "set out by the kick-start pack file", the two card pointers, "## Your final action: the minimal batch-report", "## Start here", "## Kick-start pack", and every Go-rendered card heading and pack entry line.
- "fork with a card whose member glyphs none resolved …": drop its contains needle; keep its inOrder list.
- "fork with a card naming no member glyph …": its contains list becomes "## Kick-start pack" and "no member glyphs on this card"; keep its notContains needle.
- "fork with an inline pack …" becomes "fork with an inline pack carries the inline fragments and the pack before the sandbox rule": keep the no-pack-file check; its contains list becomes the inline branch anchors "Your first turn, before any edit, reads in parallel", "The kick-start pack below lists", "which your first turn read" and `specsDir`; keep its notContains list; its inOrder list drops "on top of a broken one." and keeps the rest, the marker's own position being pinned by card 1's `TestStencils_MarkerPositions`.
- "fork with a per-card pack …": its contains list becomes "Your first work turn", "lyx loom card-pack <N>", `specsDir` and ", `lyx loom card-pack`" (the Go-chosen `card_pack_allowed` fill); keep its notContains list; its inOrder list ends at "## Start here" without the sentence after it.
- "recovery locates by quarry and carries no pack section": keep the equality with the `writePrePackBody` render; drop its contains list and the notContains needle "The kick-start pack file lists".
- "master names the fixer prompt and carries no integration fork" becomes "master names the fixer prompt": keep `testVerifyFixPromptPath`; drop "Gate findings recorded at", "## Verify gate fixes" and both notContains needles.
- "master with PATTERN inactive renders no constraints block": drop the `` `pattern/` `` count check and the contains needle; keep the notContains needles "{{", "## Constraints" and "\n\n\n\n"; drop "codebase".
- "master places the PATTERN directive ahead of the first work instruction": its inOrder list becomes "some constraints", "## Orientation".
- "fixer prompt names the gate report and the contract files": its contains list becomes `reportPath`, `testOutcomePath`, `testSummaryPath` and the Go-filled plan directory `filepath.Join(root, "_lyx", "plan")`; keep "{{" absent.

Every other row is unchanged, and `TestMasterTemplate_QuotesBulletLists`, `TestTemplates_FillRequiresEveryRequiredMarker`, `TestTemplates_Composition`, `TestRenderPrompts_FrictionDirective`, `TestRenderPrompts_ParentDirective` and `TestRenderBatchLists` stay as they are: they check Go-read lists, marker sets, composition and Go-chosen directive anchors.
Delete the six literal-copy constants the dropped needles used.

`TestBeginBatch_KickstartPack` in `beginbatch_test.go` anchors the same Go-chosen pack layouts on whole fragment sentences; its anchors shrink to the shortest phrase that tells each layout apart, the phrases `TestRenderPrompts` keeps: `quarryLocate` becomes "Locate them with one", `packFirstTurn` becomes "Read all of this in one turn", `perCardFreshRead` becomes "Your first work turn", `perCardStepOne` becomes "lyx loom card-pack <N>", and the inline rows' "Your first turn, before any edit, reads in parallel" becomes "Your first turn, before any edit"; `plainStepOne`, `packHeading`, the "## Start here" checks, the Go-rendered pack entries and the log lines stay.

`requireNotContains`'s doc comment stops claiming it pins the dropped batch-era concepts and the moved Shared Decisions: it fails the test, naming the forbidden needle, if text contains it.
The file comment heading `template_test.go` stops describing literal-statement assertions and the report-schema section: it says the file pins the master template's digest-field and outcome-key lists against webster's Go definitions, the marker sets through Fill round-trips, the composition of the fork and recovery templates, and the rendered prompts' Go-filled values and Go-chosen fragments.

**ImpactSummary:** test-only, confined to `internal/websterengine`'s `template_test.go` and `beginbatch_test.go`; the deleted constants are read only by `TestRenderPrompts`, and no production code changes.
