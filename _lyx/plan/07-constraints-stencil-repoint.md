# Card 7 — constraints-stencil-repoint

**Edit:**
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `contracts/stencils/citation_enforcement_test.go`
- `internal/websterengine/template_test.go`

**Intent:** Stop the producer stencils from naming `CONSTRAINTS.md` (D7).

Each listed stencil drops its "read `CONSTRAINTS.md` at the repo root (in full) if present" clause.
Where the sentence keeps a reason to exist, it refers instead to the PATTERN section carried in the same prompt (the injected directive), for example "follow the PATTERN entries above and existing patterns".
`webster-prefix-recovery.md`'s numbered orientation step that only read `CONSTRAINTS.md` is removed, and the list renumbered.
Read the directive's position in each template first, and phrase the reference so it reads correctly whether the optional directive rendered or not.

`citation_enforcement_test.go`'s "if present" rule is replaced by a rule that fails any stencil naming `CONSTRAINTS.md` at all.
Its unit cases prove it fails such a stencil and passes one that does not.
`TestRenderRecoveryPrompt_InstructsColdOrientation` stops requiring `CONSTRAINTS.md`.
It asserts the PATTERN-active case carries the directive (the inlined overview text of its fixture), and that the inactive case names neither `CONSTRAINTS.md` nor an orphan constraints heading.
The test's doc comment follows.

Grep the repository's stencil tests for `CONSTRAINTS.md` assertions; any other hit joins this card.

**ImpactSummary:** Rewords five stencil sentences and two tests; prompts lose a pointer to a file this task deletes.
