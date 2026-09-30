# Support log — publish-verify-after-merge

## Interview

The session ran autonomously; every turn below is a self-posed question with the auto-picked answer.

1. **What is the task?**
   Board task 12: `Publish` merges the parent into the task branch and pushes without compiling or testing.
   Incident: run `operator-surface`, PR #300, conflict-resolved merge `854eb73fe` left call sites on the old `shedrun.StatusRel(runID)` signature.
2. **Where does the work fall?**
   `internal/landingshed` (`publish.go`, `finalize.go`, `deps.go`), wired by `internal/loomcli/landingdeps.go`.
   The merge-in is `internal/mergeresolve.Resolver.Resolve`, whose `Result.AlreadyUpToDate` distinguishes a no-op merge.
3. **Is Publish the only affected producer?**
   No. `Finalize` runs the same resolver merge-in, then merges into the parent and pushes it with no reviewer.
   Auto-pick: gate both producers.
4. **Which merges trigger the gate — only conflict-resolved ones?**
   Any merge that changed the tree; a clean textual merge can still break compilation.
5. **Where does the command come from?**
   The board offers the plan's `## verify:` "or at least `go vet ./...` across build tags".
   Auto-pick: the plan's `## verify:`, told by loomcli and resolved at call time.
   The template and plan-review rubric already require it to cover every targeted package including build-tagged tests.
6. **What if the plan has no `## verify:`?**
   Auto-pick: warn and skip, mirroring webster's `ShouldRunIntegration`.
7. **What happens on failure?**
   Stuck through `stuckOrCancelled`, before any push or GitHub call (`Publish`) or parent-side merge (`Finalize`).
   The merge commit stays; the operator fixes forward.
8. **How is a resume after a failed verify kept safe?**
   On resume the merge-in is a no-op, so a tree-changed trigger alone would skip the gate and push the broken tree.
   Auto-pick: a presence-only pending-verify marker under loom scratch, written after a tree-changing merge, removed on pass, forcing the gate while present.
9. **How does the operator see what failed?**
   Output goes to a told scratch file; the Stuck reason names it with the exit code.
10. **How is the command run?**
    In-process through the platform shell in the worktree root, context-cancellable, sharing webster's shell-selection runner rather than copying it.
11. **Timeout?**
    None beyond context cancellation; `go test` bounds each test binary itself.
12. **Testing?**
    Unit tests with a faked runner and faked resolver per producer path; loomcli wiring test for call-time resolution and the drift guard; webster's existing tests pin the shared runner's unchanged behaviour.

## Rejected alternatives

- **Gate only conflict-resolved merges.**
  A clean merge can still produce a semantic conflict that fails to compile; the trigger would miss it.
- **Gate `Publish` only, as the board names.**
  Leaves `Finalize` pushing an unverified merge straight to the parent, including every no-PR landing and every parent that moved during PR review.
- **Hard-coded `go vet ./...` (plus `-tags integration`) as the command or fallback.**
  landingshed is a general producer; a Go-specific default is wrong for any non-Go repository lyx drives.
- **A landing.yaml `verify` key as the command or fallback.**
  Adds a config surface for a case the plan already covers; YAGNI.
- **Halt Stuck when the plan has no `## verify:`.**
  Harsh for a section the parser treats as optional; the plan-review rubric already flags missing coverage upstream.
- **Verify unconditionally on every `Publish`/`Finalize` pass.**
  Pays a full suite run even when the parent has not moved and webster's integration already verified the tree.
- **Record the last verified HEAD SHA instead of a presence marker.**
  More robust against the pre-marker crash window, but adds state interpretation for a one-file-write-wide window.
- **Abort or reset the merge commit on failure.**
  Discards a conflict session's work; the fix belongs on the merged tree.
- **Spawn an LLM fix-forward session on failure.**
  Out of the board's direction (halt stuck); a human is the fixer for rows with no `on_stuck`.
- **Instruct the conflict-resolution session to run the verify itself.**
  Prompt-level and unenforced, and it never covers clean merges.
- **Copy webster's `runVerifyCommand` into landingshed.**
  Two shell-selection copies drift.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Gate Finalize too?** Resolved, auto-pick: yes. Same resolver, same failure, no reviewer downstream.
- **Q2 — Trigger condition?** Resolved, auto-pick: `!AlreadyUpToDate`, or the pending marker present.
- **Q3 — Command source?** Resolved, auto-pick: plan `## verify:`, told by loomcli at call time.
- **Q4 — Missing `## verify:`?** Resolved, auto-pick: warn and skip.
- **Q5 — Failure disposition?** Resolved, auto-pick: Stuck, merge commit kept, fix forward.
- **Q6 — Resume safety?** Resolved, auto-pick: presence-only pending marker under loom scratch, shared by both producers.
- **Q7 — Output visibility?** Resolved, auto-pick: told scratch output file named in the Stuck reason.
- **Q8 — Runner?** Resolved, auto-pick: share webster's shell-selection runner, extended with context and an output writer.
- **Q9 — Timeout key?** Resolved, auto-pick: none.
- **Q10 — Spawn failure: error or Stuck?** Resolved, auto-pick: Stuck, so the reason reaches the operator's envelope.
- **Environment:** no tool failures observed during exploration.
