```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I traced what calls, reads or asserts it and whether the diff leaves any of it wrong.

**Per change, the affected set and whether it is covered**

1. **`internal/boardcli/text_test.go` deleted.** This removes `TestRenderCompact` and its helper `strPtr`.
   - `strPtr` is still used by boardengine tests, but those use their own copy at `internal/boardengine/runlock_test.go:20`. Nothing in boardcli is left calling the deleted helper. Covered.
   - `RenderCompact`'s callers, `cli.go:946` and `cli.go:1183`, are unchanged. Covered.
   - `docs/benchmarks/test-redundancy.md:267` names `TestRenderCompact`. That file is a measurement report, which the prose rules exempt from going stale. Covered.
   - The `docs/research/...` files that name it are recorded bench data. Covered.
   - No stencil or spec reads `lym board list --text`: every stencil reference is to plain `lym board list`, the JSON output. Covered.
   - **Not covered:** `internal/boardcli/verbs_test.go:347-352`, `:359` and `:366` still compare `list --text` and `find --text` output byte for byte, column padding included. That is the same layout of the same human-read output that the commit deleted `TestRenderCompact` for. This is finding F1.

2. **README, project-file and design-doc goldens and their helpers deleted from `render_test.go`.** The helpers are `readmeFixture`, `readmeTypes`, `projectSummary`, `rootProjectSummary`, `emptyProjectEntries`, `emptyEntriesAtLevelSix`, `rootEntriesWithProjects`, `ownEntriesHeading` and `getKeys`.
   - Outside the deleted text, no `_test.go` file references any of them. Covered.
   - `stringPtr`, which `view_test.go` uses, is defined at `layer_test.go:331`, so it survives the deletion. Covered.
   - The `TestRenderOrder` keep-reason at `layer_test.go:199` named "the render goldens". The diff updates it. Covered.
   - `docs/benchmarks/test-redundancy.md:292` pairs `TestRenderOrder` with `TestRenderReadmeGolden`. It is a measurement report and exempt. Covered.
   - The file header was rewritten to describe what the file keeps. Covered.

3. **The `BoardDataFile` existence check removed from `TestRenderToDiskManifestCleanup`.** The loop at `render_test.go:424-431` still checks that `BoardDataFile` is written and that the manifest lists it. This is a duplicate removed, not a loosened check. Covered.

4. **The heading parsing in `view_test.go` replaced with literals.**
   - The helpers `readmePlacement`, `readmePlacements`, `tableCells`, `rootSectionNames` and their regex vars have no other user. Covered.
   - The `regexp` import was dropped. Covered.
   - The `TestRenderBoardData` doc comment no longer says "as the README shows it". Covered.
   - The comments in `viewFixture` and the projects subtest still mention the README. They describe README order, which still holds. Covered.

5. **The `Discussed` flag check moved into the `RenderToDisk` subtest of `TestRenderBoardData`.** The doc comment of `RenderToDisk` in `render.go:41-44` states this contract, and the moved check pins it. Covered.

6. **The repository's rule documents.**
   - The `golden` mentions in `pattern/PATTERN-sole-parsers.md:38` and `contracts/stencils/darn/darn-fragment-make-change-gated.md:25` are about parser fixtures and test-run values, not human-read layout. Neither is made stale. Covered.
   - No PATTERN change is declared, and none is needed. Covered.
   - The range's commit carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. The only removed check outside the goldens is the duplicate in item 3, so no trailer is needed.

**Findings**

- **F1.** MEDIUM, class `scope`.
  - Location: `internal/boardcli/verbs_test.go:347-352` (also `:359` and `:366`).
  - Summary: the sweep left byte-exact layout goldens of `lym board list --text` and `find --text` in `stepListAndFindText`. They pin column padding, such as `"T1  task     0m  0m  soon   Soon thing ..."`, for output the commit message and `_lym/landing/summary.md:18-20` call human-read and parsed by nothing.
  - Fix: keep what other code relies on, which is that the root row is placed before the first project, note or done entry, that the label filter keeps only matching entries, and that `find` matches. Assert those through the JSON `list` output or by row prefix, and drop the exact padded-line equality.
  - Evidence: `internal/boardcli/verbs_test.go:347-352`; the commit message of c9cadd649; `internal/boardcli/text.go:1`.
  - This overlaps the scope lens, since it is the record's "sweep the repository" goal.

**Tool calls**

1. A grep for `stringPtr` and `strPtr` definitions and uses, golden wording in the PATTERN and contract docs, and the benchmark doc rows. My context lacked where the surviving helpers are defined and whether any rule document names the deleted tests.
2. A grep for the `stringPtr` definition site and the golden wording in the two matched rule docs. The first grep's output was cut short by `head`, and the matched lines needed reading.
