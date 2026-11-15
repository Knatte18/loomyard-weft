```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, listing each changed function or branch, the case run, and whether it is correct.

**List**

The diff changes no Go function, branch or error path. It touches three prose files: `contracts/stencils/edit/edit-directive.md`, and the package doc comments in `internal/editdirective/doc.go` and `internal/fabricengine/doc.go`. So there is no code to run step by step.

- `editdirective.Directive` (renders `edit-directive.md`): the case run is the new three lines, rendered unconditionally.
  - The text has no `{{` marker, and the file still starts with its `<!--` banner, which `Directive` strips (the test checks no leading comment).
  - The new lines sit inside the body, so the trailing-newline and marker-free checks in `editdirective_test.go:16-37` still hold. Correct.
- `TestStencils_NoEditRuleRestatement` (`parentdirective_test.go`): the case run is the new sentences in `edit-directive.md`.
  - That test skips the `edit-directive` stencil by name (`parentdirective_test.go:163`), so the new text cannot trip it.
  - Other stencils are unchanged. Correct.
- Claim "`.scratch` is a link to a directory kept for your task alone", checked against `fabricengine/doc.go:649`.
  - The link target is `<weft worktree>/<anchor>/.lym/scratch`, inside the pair's own weft worktree. "Your task alone" holds. Correct.
- Claim "a build under `.scratch/` resolves a relative reference from the link's target, not from the working directory", checked against `fabricengine/doc.go:655-656` (target in the weft worktree, not the warp anchor).
  - The directive and both doc comments agree. Correct.
  - This describes how a tool resolves a symlinked path. It is not behaviour of this code, and I did not run a build to confirm it.

**Findings**

None.

**Tool calls:** none. My context already held everything this lens needed.
