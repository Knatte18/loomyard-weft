```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set** — what the diff's changes force outside the lines it touches.

**Per change, the set found:**

1. **README project sections now render as `&lt;details open&gt;` blocks.**
   - Production readers or parsers of README headings: none found. The only caller of `renderReadme` is `Render` (`render.go:118`); `boardtest/bench_test.go:48` calls `Render` but doesn't assert on the heading shape. Covered.
   - Docs: `doc.go:313` and `docs/overview.md` were updated. `render.go:4` and the `renderReadme` comment were updated. The test-doc comment on `TestRenderReadmeGolden` was updated. Covered.
   - Stencils and specs: my grep found none that describe README `##` sections. Covered.
2. **`projectLink` split into `projectLink` and `projectFile`.** `upLink` and `projectSection` still call `projectLink`, with the same output. Covered.
3. **`projectSection` split into `projectSection`, `openMembers` and `projectSectionContent`.**
   - The nested-section caller `projectBody` (`render.go:589`) still goes through `projectSection`, with the same heading and content. Covered.
   - The root's member rule (`render.go:554`) is unchanged. Covered.
4. **PATTERN entries and commit messages.**
   - Declared PATTERN changes: none declared, and none needed.
   - No `Pattern-Departure:` trailer is present or needed.
   - The commit message matches the diff.
   - I found no PATTERN entry the change touches.

**Findings:** none.

**Tool calls (1 of 6):**
1. Grep across `.go`, `.md`, `.tmpl` and `.yaml` for other readers of README heading shapes and callers of `renderReadme` and `Render`. My context lacked any repo-wide search for these.
