# Card 11 — orch-docs

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`
- `CLAUDE.md`
- `contracts/specs/refusal-spec.md`

**Edit:**
- `contracts/stencils#wordingClaims`
- `internal/orchcli/doc.go`

**Intent:**
Each landing step moves to its new owner in the texts the orch and the operator read (D7, D8, D9).

`contracts/stencils/orch/orch-template-role.md`:
- `## The run loop` item 2 names the one exception to batten never resuming a halted child: batten sends the first Publish verify failure of a run back to the failure record's target and resumes it itself; every other halted child is still resumed with `lyx loom resume` in the task worktree once its cause is known.
- `## The run loop` item 4 sends a `[batten notice]` naming newly ready tasks to `## After landing` and every other `[batten notice]` to Investigating a stop, and `## Investigating a stop`'s first lines exclude the ready notice beside the parent-review request, so a ready notice is never investigated as a stop.
- `## After landing` is replaced: a landing needs nothing from the orch, since batten's Land row pulls the prime, runs the hub's `deploy` command, prunes the board and then tears the pair down;
  a `[batten notice]` naming a Land or Worktree-Teardown failure is a stop, handled under Investigating a stop;
  a notice naming newly ready tasks is the cue to start them with `lyx batten run <slug> --window`;
  while the hub's landing `deploy` key is empty, the orch runs `update-plugins.sh` by hand on a ready notice or at its next wake after an approval;
  drive reports are still read from the prime's own copies only when the run had trouble, and friction notes are still never read.
  The section no longer tells the orch to pull, deploy, reconcile, prune or check the teardown after a landing.
- `## PR-Gate`: after approving, the orch amends the board entries the run superseded; batten resumes the child after `lyx loom approve` or `lyx loom reject`, so the orch runs no `lyx loom start` loop after either.
- The GitHub inbox is taken onto the board with `lyx board intake` at PR-Gate and at any other wake, since triage needs judgement; this goes into `## Where a finding goes`.
- A hub config edit's fabric commit is the orch's whenever it makes one, not a landing step.
- `## Acting without asking` drops "land and clean up" from the orch's own actions, which batten's Land and Worktree-Teardown rows now take, and names handling a Land or teardown failure notice instead.
- `## Escalations from a child`: the Publish verify-failure line says batten retries the first such failure itself, so a notice for one means the second failure, which goes to Investigating a stop; the `lyx loom goto --to Webster-Burler` command stays named there as that stop's way forward.

`CLAUDE.md`'s `## Production` standing exception says batten's Land row runs the hub's `deploy` command after a landing, and the hub orch runs `update-plugins.sh` itself only after its own pushed fix, on a clean `main`, and says so in its report.

`contracts/specs/refusal-spec.md`'s "Publish: verify failed" row adds to its way forward that a batten run watching the child takes it once on its own, sending the child to the record's target producer and resuming it, so the operator's goto and resume are for the second failure or an unwatched run.

`internal/orchcli/doc.go`'s run-loop description follows the stencil: step 5 no longer has the orch pull, deploy, reconcile or remove the pair after landing, but says batten's Land row lands the run and the orch acts on a Land or teardown failure notice or a ready-task notice, and deploys by hand on a ready notice or at its next wake after an approval while the hub's landing `deploy` key is empty;
step 4 says a Publish blocked on a failed verify reaches the orch only on its second failure in the batten run, since batten retries the first itself;
step 3 drops the `lyx loom start` after `lyx loom approve`, since batten resumes the child.

`wordingClaims`' row for `orch-template-role.md` drops the `## After landing` claim on closing a batten run by hand and gains claims, each in its section:
`## The run loop` names batten's own retry of the first Publish verify failure as the exception, beside the kept claim that a halted child is resumed with `lyx loom resume` in the task worktree, and sends a ready notice to `## After landing`;
`## Investigating a stop` excludes the ready notice from what a stop is;
`## Acting without asking` holds no `land and clean up`;
`## After landing` says a landing needs nothing from the orch, names the ready notice's `lyx batten run <slug> --window`, and holds none of `lyx fabric pull`, `lyx config reconcile` and `lyx board prune`;
`## PR-Gate` says the orch runs no `lyx loom start` loop after an approval;
`## Escalations from a child` says batten retries the first Publish verify failure;
the existing claims on the drive reports, the friction notes and the Webster-Burler goto stay.

**ImpactSummary:** stencil text and its claim rows, CLAUDE.md, one refusal-spec row and orchcli's package doc; no Go behavior changes.
