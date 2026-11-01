```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens — what the `projectFacts` change forces outside the touched lines.

**Per-change list**
- `projectFacts` (`render.go:346`) now calls `slugCell`. Set found: its two callers, `fileHeader` (`render.go:334`, project files) and the README section at `render.go:381`.
  - Both callers go through `b.out.DesignPrefix`, which is the same prefix `renderDesigns` uses (`render.go:111`).
  - Both are covered.
- The design file is written when `e.Body != ""` (`render.go:759`), and `slugCell` links on the same test (`render.go:616`). The conditions match, so a body-less project gets no dangling link. Covered.
- Docs:
  - `doc.go:176` is updated.
  - The `render.go:344` comment is updated.
  - The `render_test.go:274` test doc is updated.
  - Other docs mentioning a facts line are `doc.go:175` and `doc.go:180`, `render.go:4`, `render.go:328` and `render.go:369`. None of them describes the slug form, so none needs the change. Covered.
- Goldens: the only test fixture with a project that has a `Body` is the `ship` row at `render_test.go:481`. Its README golden (line 510) and project-file golden (line 641) are both updated. No other test or golden renders a project with a body, so none is stale. The same fixture still holds `tidy` and nested `inner`, which have no body and keep a plain slug. Covered.
- PATTERN.md and `pattern/`: the diff touches neither, and the focus names no declared changes. No entry covers board rendering. Nothing to land.
- Commit message: it carries no `Pattern-Departure:` trailer, and none is needed because no PATTERN entry is departed from.
- Stencils: none changed.

**Findings:** none.

Tool calls (2):
1. `grep` for `KindProject` fixtures with `Body:` and for the facts-line text in tests, docs and goldens, because the manifest did not show other fixtures.
2. `grep` for "facts" mentions in docs and code, plus `git diff --stat` on `PATTERN.md` and `pattern/`, to find other descriptions of the facts line and any PATTERN changes.
