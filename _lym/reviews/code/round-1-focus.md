---
round: 1
focus:
  - "internal/samplerengine/spec.go SessionSpec: does the `pattern.Directive` call match how `VerifyTable` renders the implementer seat's directive (argument order, the worktree root and stencils dir passed, the trailing nil), and does it return empty without error when the worktree has no PATTERN overview, as the optional marker assumes?"
  - "internal/samplerengine/spec.go: the board entry asks for a marker added to the stencil and the directive rendered with `pattern.RoleImplementer`; settle whether making `pattern_directive` optional via `stencil.FillOptional` is licensed, or whether every other role's stencil requires the marker non-empty, and whether the optional choice drops a rule the other roles keep."
  - "contracts/stencils/sampler/sampler-template-agent.md: is `{{.pattern_directive}}` placed where the other code-writing roles' stencils place the directive, does it stay a top-level substitution with no conditionals, and does the header comment's marker list and 'paths only' claim still read true?"
  - "internal/samplerengine/spec_test.go: the new subtest covers a worktree with an overview; does an existing test already cover the no-overview path (empty directive, prompt still fills), and does the new test assert the prompt carries the directive rather than restate `pattern.Directive`'s own behavior?"
  - "Scope: the diff touches only the sampler engine, its stencil, its doc and its test; check that no other stencil copy, seeded default or doc the repository's constraints name for a stencil marker change is left out of the diff."
  - "Commit message and doc comments: the commit body and the `doc.go` and `spec.go` comments claim the directive renders from the task worktree; verify the claim against the code, and check the comments against the repository's comment conventions."
---

Seed focus for round 1. The answer key is the decision record's `## Goal` alone; its other sections are empty.
