# Card 3 — pattern-designer-role

**Edit:**
- `internal/pattern/pattern.go`
- `internal/pattern/pattern_test.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/discussiontemplate_test.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine/prompt.go`
- `internal/loomengine/discussion_test.go`

**Create:**
- `contracts/stencils/pattern/pattern-directive-designer.md`

**Intent:** Give Discussion-Write the PATTERN directive through a new `RoleDesigner` (D5).

Add `RoleDesigner` to `internal/pattern`'s `Role` set, with its own stencil-name constant and `Directive` case, mirroring `internal/friction`'s `RoleInterview`.
The new stencil follows card 2's shape: a banner naming its consumer and its one `{{.pattern_overview}}` marker, then an instruction section worded for design.
Every decision complies with the entries, or names the entry it changes and why.
An entry cannot be waived silently.
Background files are read under the literal `pattern/` where they touch a decision.
Register the stencil in `stencils.go`'s embed vars and registry table beside the three existing pattern rows.

`loom-template-discussion.md` gains an optional `{{.pattern_directive}}` marker placed before the interview's first work instruction.
`composePrompt` and `DiscussionSpec` fill it via `stencil.FillOptional`, keeping it optional exactly as Plan-Write's template does.
`DiscussionSpec` calls `pattern.Directive(layout.WorktreePath(), stencilsDir, pattern.RoleDesigner)`, and a directive error is returned, as in `PlanSpec`.

Tests:
- `pattern_test.go` adds the designer role to its every-role cases.
- `discussion_test.go` proves the rendered prompt carries the directive when `PATTERN.md` exists at the worktree root and omits it when it does not.
- `discussiontemplate_test.go` (and any marker pin it holds) admits the new optional marker.

`internal/pattern/doc.go`'s role section is rewritten in card 4, once both new roles exist.

**ImpactSummary:** Adds one Role value and one stencil; Discussion-Write's prompt gains an optional section; no other directive caller changes.
