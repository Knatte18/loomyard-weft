```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff 71d6ee1a..HEAD.

List:
- `contracts/stencils/engine/claude/edit-directive.md:9`: no function or branch. Case run: `editdirective.Directive(stencils, claude)` reads the file and renders it as the `edit_directive` marker value. The new line adds no `{{` token, so the copy stays marker-free, as its header comment requires (lines 1-4). It renders the same way as before. Correct.
- `contracts/stencils/loom/loom-rubric-discussion-review.md:43-46`: no function or branch. Case run: ReadRubric fills the rubric's markers and `StripLeadingComment` removes the header comment. The new bullet adds no marker; the only markers remain `{{.stencils_dir}}`, `{{.specs_dir}}` and `{{.amendment}}`. It sits after the header comment, so it is never stripped. Correct.
- `internal/editdirective/doc.go:13`: comment only, with no executable code changed. Correct.

Findings: none. The diff adds or changes no function, branch or error path, and no changed text introduces a marker or template token that would alter how it renders or composes.

Tool calls: none.
