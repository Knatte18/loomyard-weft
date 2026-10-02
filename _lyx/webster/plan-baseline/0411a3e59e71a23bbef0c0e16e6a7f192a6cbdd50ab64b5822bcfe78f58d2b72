# Card 13 — The hub reviews through a one-shot fork

**Prosa:**
- `contracts/stencils/orch/orch-template-start.md`
- `contracts/stencils/orch/orch-template-resume.md`

**Edit:**
- `internal/orchengine/prompt_test.go`

**Intent:**
Give the hub decision D12's handling of a review-request notice.

`orch-template-start.md` gains one paragraph: a message naming a parent-review request is handled by a one-shot fork that reads the request's brief, reviews, and submits with the brief's `lyx loom review` command, so the hub's own context grows by the notice and the fork's summary only; a repeat notice for a request already forked for starts no second fork.
`orch-template-resume.md` gains the same instruction as one more clause on its single line, since that stencil must render to one line.

`prompt_test.go` asserts both rendered prompts name `lyx loom review` and the one-shot fork, and the resume prompt stays one line.

**ImpactSummary:** Prompt text for the orch start and resume prompts and their tests; no marker changes, so `orchengine`'s render functions are untouched.
