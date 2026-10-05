# Card 17 — The orch role stencil describes the loop batten runs

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`

**Uses:**
- `contracts/recipes/batten-recipe.yaml`
- `contracts/recipes/loom-recipe.yaml`
- `internal/battenshed/doc.go`
- `internal/landingshed/doc.go`
- `internal/shedrun#DriveReportsDir`
- `internal/loomengine#LoomFrictionDir`
- `internal/battencli/cli.go`

**Intent:**
Finding F1: the role stencil is the orch's only procedure, and it misstates the run loop.
Rewrite "The run loop", "PR-Gate" and "After landing" to describe the loop as batten runs it, then check every other section against `batten-recipe.yaml` and `loom-recipe.yaml` the same way.

- The run loop: `lyx batten run <slug>` from the prime, detached with `setsid nohup` until `lyx batten start` exists, creates the task pair (Worktree-Create, Seed-Child), drives the loom run inside it (Run-Shed) and tears the pair down after the run ends (Worktree-Teardown).
  The `lyx fabric add <slug>` step goes.
  A halted child (blocked, paused or failed) is a wait batten never resumes on its own: the orch resumes it with `lyx loom start` in the task worktree.
  `lyx batten pause <slug>` and `lyx batten goto <slug> --to <producer>` leave the batten run paused (`internal/battencli/cli.go`), so after either the orch resumes it with `lyx batten run <slug>`, which starts or resumes the lifecycle run.
- PR-Gate: approve with `lyx loom approve` or reject with `lyx loom reject <review-file>` in the task worktree; batten reads the decision and resumes the child itself, so the "start the run again so it finalizes" step goes.
  Finalize syncs the task branch with main, squashes it onto main, marks the board task done, pushes main and closes the pull request with a comment naming the landing commit, so the pull request ends closed, not merged.
- After landing: the `lyx fabric remove --remote <slug>` step goes, since batten's teardown removes the pair.
  What remains is pulling in the prime, reading the friction notes and the driver's drive reports from the prime's own copies, and amending the board entries the run superseded.
  The copies are the friction notes directory (`internal/loomengine#LoomFrictionDir`) and the run's `drive-reports` directory (`internal/shedrun#DriveReportsDir`), both under the prime's durable lyx directory, which the landing carries there; the stencil says "the prime's own copies" and spells no `_lyx` path, per `PATTERN-lyxdirs-single-declarer`.
  A step that reads from the removed task worktree goes; none of the current steps does once the reading names the prime's copies.

- Escalations from a child: checked against the recipes, the one other section that misstates a step.
  The `lyx loom circling continue` and `lyx loom circling accept` verbs only record a decision file and resume nothing (`internal/loomcli/circling.go`), and batten never resumes a halted child, so after either the orch runs `lyx loom start` in the task worktree to resume the run.
  `lyx loom goto` and `lyx shed goto` leave the run paused (`internal/loomcli/cli.go`, `internal/shedcli/cli.go`), so after a goto the orch runs `lyx loom start` in the task worktree as well.
  The design call is recorded with `lyx loom decision add`, not `lyx loom decision`.
  The closing "message the driver by name, telling it to resume" stays only for a driver that is still alive; it no longer stands in for the resume.
- Parent-review, The board, Where a finding goes, Acting without asking, Status reports, Messaging and Which role can do what name no recipe step, so the check changes nothing there.

A `contracts/stencils/claims_test.go` row for `orch-template-role.md` in `wordingClaims` pins the corrected facts:
must claims for the batten run creating and tearing down the pair, for batten resuming the child after a decision, for `lyx loom reject <review-file>`, for Finalize marking the board task done, pushing and closing the pull request, for reading the friction notes and the driver's drive reports from "the prime's own copies" after landing, for `lyx loom start` resuming after a circling decision, for `lyx loom start` resuming after a goto, for `lyx batten run` resuming after a batten pause or goto, and for `lyx loom decision add`;
mustNot claims for `lyx fabric add`, `lyx fabric remove`, `squash-merges` and `start the run again`.

**ImpactSummary:** orch role stencil prose plus one new row in the stencil claims table; no Go behavior changes.

**Commit:** 17: orch: the role stencil describes the loop batten runs
