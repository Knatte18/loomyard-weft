# Card 17 — the orch's after-landing routine closes batten

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`

**Intent:**
Make the orch close a batten run that ended before its teardown (decision D11), whatever stopped it.

`orch-template-role.md`'s `## After landing` section gains a first step, before "Batten's teardown removes the pair.": read `lyx batten status <slug>`; when its state is not done, run `lyx batten run <slug>` from the prime and confirm with `lyx batten status <slug>` that it reaches done, which runs Worktree-Teardown.
The stencil keeps one sentence per line, tells the orch nothing to ask the operator (`PATTERN-parent-directive`), and is read at call time from the committed stencil (`PATTERN-stencil-ownership`).

Test: `claims_test.go`'s `orch-template-role.md` entry gains a claim in section `## After landing` that the section names `lyx batten status <slug>` and `lyx batten run <slug>`, with its why ("after landing the orch closes a batten run that stopped before its teardown").

**ImpactSummary:** One stencil section and one claim; no Go behaviour changes.
