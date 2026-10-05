# Card 17 — The orch role stencil describes the loop batten runs

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`

**Uses:**
- `contracts/recipes/batten-recipe.yaml`
- `contracts/recipes/loom-recipe.yaml`
- `internal/battenshed/doc.go`
- `internal/landingshed/doc.go`

**Intent:**
Finding F1: the role stencil is the orch's only procedure, and it misstates the run loop.
Rewrite "The run loop", "PR-Gate" and "After landing" to describe the loop as batten runs it, then check every other section against `batten-recipe.yaml` and `loom-recipe.yaml` the same way.

- The run loop: `lyx batten run <slug>` from the prime, detached with `setsid nohup` until `lyx batten start` exists, creates the task pair (Worktree-Create, Seed-Child), drives the loom run inside it (Run-Shed) and tears the pair down after the run ends (Worktree-Teardown).
  The `lyx fabric add <slug>` step goes.
  A halted child (blocked, paused or failed) is a wait batten never resumes on its own: the orch resumes it with `lyx loom start` in the task worktree.
- PR-Gate: approve with `lyx loom approve` or reject with `lyx loom reject <review-file>` in the task worktree; batten reads the decision and resumes the child itself, so the "start the run again so it finalizes" step goes.
  Finalize syncs the task branch with main, squashes it onto main, marks the board task done, pushes main and closes the pull request with a comment naming the landing commit, so the pull request ends closed, not merged.
- After landing: the `lyx fabric remove --remote <slug>` step goes, since batten's teardown removes the pair.
  What remains is pulling in the prime and amending the board entries the run superseded; drop any step that reads from the removed task worktree.

A `contracts/stencils/claims_test.go` row for `orch-template-role.md` in `wordingClaims` pins the corrected facts:
must claims for the batten run creating and tearing down the pair, for batten resuming the child after a decision, and for Finalize closing the pull request;
mustNot claims for `lyx fabric add`, `lyx fabric remove` and `squash-merges`.

**ImpactSummary:** orch role stencil prose plus one new row in the stencil claims table; no Go behavior changes.

**Commit:** 17: orch: the role stencil describes the loop batten runs
