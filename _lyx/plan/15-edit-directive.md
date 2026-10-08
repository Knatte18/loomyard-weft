# Card 15 — The shared edit directive and its renderer

**Create:**
- `plan:internal/editdirective#MarkerName` -> `const MarkerName = "edit_directive"`
- `plan:internal/editdirective#Directive` -> `func Directive(stencilsDir string) (string, error)`
- `plan:contracts/stencils#EditDirective` -> `var EditDirective []byte`
- `internal/editdirective/doc.go`
- `internal/editdirective/editdirective_test.go`
- `contracts/stencils/edit/edit-directive.md`

**Edit:**
- `contracts/stencils#entries`

**Uses:**
- `internal/parentdirective#Directive`
- `internal/stencilstore#Read`

**Intent:**
Decision 14: one shared stencil states the no-script edit rule, and one Go renderer reads it; cards 16 to 20 fill it into every spawned role's opening stencil, and card 21 pins that.

`contracts/stencils/edit/edit-directive.md` is a marker-free stencil (registry name `edit-directive`).
Its leading comment says `internal/editdirective.Directive` renders it unconditionally into the `edit_directive` marker of every spawned role's opening stencil, and that it is the rule's only statement.
Its body is one `## Editing files` section stating the rule in the wording of the shuttle python deny's steer (`steerPythonDeny` in `internal/shuttleengine/claudeengine/settings.go`):
- Change files only with Edit or Write, never with a script that rewrites them: not `sed`, `awk` or `perl`, and not an inline `python` or `node` heredoc.
- A bulk change is one Edit per site, or `replace_all` for an identical string.
- Never use `sed` at all, even to read; search and read with `grep`, `awk` or `cat`.
`stencils.go` embeds it as `EditDirective` and registers it in `entries` beside the parent-directive rows.

`internal/editdirective` mirrors `internal/parentdirective`.
- It imports only the standard library, `stencil` and `stencilstore`, and derives no path (`PATTERN-stencil-ownership`).
- `MarkerName` is `edit_directive`.
- `Directive` reads `edit-directive` from the told `stencilsDir` through `stencilstore.Read` at call time, strips the leading comment and trailing newlines, and returns the text.
- A failed read returns `("", err)` naming the stencil.
- Unlike `pattern.Directive` it has no inactive case: the text renders in a repo without `PATTERN.md` as in one with it.
- `doc.go` says what the directive is, who renders it, where it is filled, and that it constrains the tool a role edits with, never what the role may write.

Tests in `editdirective_test.go` (untagged, over `stencilkit.Seed`):
- `Directive` returns text containing `Edit or Write`, `sed`, `replace_all` and `grep`, with no `{{` and no leading comment.
- With the stencil removed through `stencilkit.Remove`, it returns an error naming `edit-directive`.
`registry_test.go`'s existing tree-versus-registry check covers the new file's registration unchanged.

**ImpactSummary:** One stencil registration added to `entries`; no stencil carries the marker yet and no renderer calls `Directive` yet.

**Commit:** 15: editdirective: one shared statement of the no-script edit rule and its renderer
