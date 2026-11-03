## Goal

```markdown
## Seen (2026-11-02)

Run `cold-gate-fixer` ran on osl-1046 and its work reached `main` (`a9536680`), but its batten run never finished Land and Worktree-Teardown there.
After `lym adopt root` took the hub over to this machine, the run stood `paused` at Run-Shed with pause reason "pause requested", with its task worktree and its loom-status terminal still up, for hours.
No notice reached the orch, and dobby does not resume a paused run, so nothing finished it; the operator noticed the open terminal.
`lym batten cancel cold-gate-fixer --remove` refused, saying the work had reached its parent, and named `lym batten run cold-gate-fixer`, which then finished Land and Worktree-Teardown at once.

Whether the pause came from the adopt hold or from the orch pausing the adopted runs is not known.

## Idea

- A run whose work already reached its parent has nothing left to decide: its remaining rows are Land and Worktree-Teardown.
  Dobby finishes such a run even when it is paused, or at least sends the orch a notice naming it.
- `lym adopt` lists the adopted runs whose work already landed and finishes them as part of the takeover.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): A landed paused run under a pause entry waits for the lift; adopt needs no step of its own

Decision: Keep the writer's scope. A paused run whose work reached its parent and that a tender pause entry covers gets no pass and no notice while the entry stands: the full pass at the lift finishes it, as planPass already gives. lym adopt gains no step: after a takeover, the adopting machine's dobby finishes such a run by the same paused-and-reached-parent pass once no pause covers it, which is what the Goal's second idea asked for.

Rationale: A pause entry is the operator's or hibernate's hold on all work, and Land pushes and tears down, so finishing under it would break the hold. The incident's run stood paused with no covering entry, which the new pass now finishes; making adopt finish runs itself would duplicate dobby's pass.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
