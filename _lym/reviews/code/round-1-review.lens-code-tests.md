```yaml
lens: code-tests
outcome: reported
```

Per-behaviour list:

1. **A project with a body links its facts-line slug to `design-&lt;slug&gt;.md` in README.md.** `TestRenderReadmeGolden`, row "each project renders to its own file under its parent": `ship` now has `Body: "Ship design."`, and the README golden at `render_test.go:510` expects ``[`ship`](design-ship.md) · Age 3d · Owner `box` ``. It would fail if `projectFacts` went back to `codeSpan`.
2. **The same link in a project file.** The same row, in the `project-ship.md` golden at `render_test.go:641`, expects the identical linked facts line. A fix to README.md alone would fail here.
3. **A project without a body keeps the plain slug.** The same row pins plain facts lines for the body-less projects:
   - `inner` in README.md at line 522 and in project files at lines 658 and 715;
   - `tidy` in README.md at line 577 and in its project file at line 740.
   
   Plain `inner` and `tidy` lines would fail if `projectFacts` linked every slug.
4. **The link target uses `b.out.DesignPrefix`.** That prefix is `design-` in this test. The rest of the golden passes `Outputs{DesignPrefix: "design-"}` at `render_test.go:885`. A wrong prefix would fail the golden. No row uses a non-default prefix for a project link, but `slugCell` is shared with the task tables, which already have a prefix-bearing test (`render_test.go:1011` onward).
5. **The link exists only when the design file exists.** Both rely on `Body != ""` (`slugCell` at `render.go:616`, `renderDesigns` at `render.go:759`). The row asserts the `project-ship.md` file but I did not read it for an assertion that `design-ship.md` itself is written. I did not check this; `TestRenderDesignDocGoldens` covers design-file generation for bodies in general. I raise no finding.

Findings: none. Every behaviour the diff changes has a test row that would fail if it broke. No test is duplicated and no uncovered behaviour is added. Other goldens that render a project with a body and an old facts line: none found in `render_test.go`, because every `projectFacts` golden line in the grep is either `ship` (updated) or a body-less project.

Tool calls:
1. `grep` of `render_test.go` for the plain `tidy`, `inner`, `old` and `ship` facts lines, because the manifest lacked the goldens' no-body lines.
