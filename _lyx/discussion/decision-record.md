# Publish: verify the tree after merging the parent

## Goal

Stop the landing producers from pushing or landing a tree that a parent merge-in broke.
Today `Publish` merges the parent into the task branch through `internal/mergeresolve` and pushes without compiling or testing the result.
In run `operator-surface` (PR #300) the conflict-resolved merge `854eb73fe` left test call sites on the old `shedrun.StatusRel(runID)` signature;
`go build` passed, `go vet`/`go test` failed, and only the hub's PR review caught it.
After this task, every parent merge-in that changes the task tree is followed by the plan's `## verify:` command, and a failure halts the run as Stuck before anything leaves the worktree.

## Scope

In:

- A post-merge verify gate in `internal/landingshed`, run after the parent merge-in in both producers:
  `Publish` (before the push and the pull-request create/refresh) and `Finalize` (before the parent-side merge, the board update and the parent push).
- A pending-verify marker so a resumed run re-verifies a merge it has not yet verified, even when the next merge-in is a no-op.
- `internal/loomcli` wiring that tells landingshed the verify command (the plan's `## verify:` line), the marker path and the output-file path.
- Reuse of the in-process shell-command runner webster already has, so shell selection lives in one place.
- Docs: `internal/landingshed/doc.go`'s package doc, and the `## verify model` section of `contracts/specs/loom-plan-spec.md` naming landing as the verify line's second consumer.

Out:

- Any automatic fix attempt after a failed verify (no LLM session, no bisect).
- Rolling back or aborting the merge commit on failure.
- A fallback verify command (a landing.yaml key, or a hard-coded `go vet`) when the plan has no `## verify:` section.
- A verify timeout config key.
- Changes to the conflict-resolution stencil, to webster's own integration stage, or to the approval path.

## Decisions

- **Trigger: any merge-in that changed the tree.**
  The gate runs when `mergeresolve.Result.AlreadyUpToDate` is false, whether the merge concluded cleanly or through a conflict session.
  Rationale: a clean textual merge can still break compilation (a semantic conflict); only a no-op merge leaves the tree webster already verified.
- **Both landing producers carry the gate.**
  Rationale: `Finalize`'s merge-in has the identical failure mode and is worse — it lands on the parent and pushes with no reviewer downstream, both on no-PR (task-to-task) landings and when the parent moved while an approved PR sat in review.
- **Command source: the plan's `## verify:` section.**
  `internal/loomcli` reads it through `internal/planparser` (the already-chained single command line webster runs) and tells it to landingshed; landingshed never reads or names the plan.
  The value is resolved at call time, not at producer construction, because producers are constructed before the plan exists on a fresh run.
  Rationale: the plan's verify is repo-agnostic, and the plan template plus plan-review rubric already require it to cover every targeted package including build-tagged tests.
- **No command means no gate.**
  An empty verify command skips the gate with a `logger.Warn` naming the skip; the producer then proceeds as today.
  Rationale: mirrors webster's own `ShouldRunIntegration` skip, and landingshed must not embed a language-specific default.
- **Failure halts as Stuck, through the producer's existing `stuckOrCancelled` path.**
  A non-zero exit and a spawn failure (the shell cannot start) both return Stuck with a reason naming the parent branch, the exit code (or spawn error) and the output file path.
  `Publish` makes no push and no GitHub call; `Finalize` performs no parent-side merge, no `MarkTaskDone`, no parent push, no PR close.
  Rationale: neither row has an `on_stuck` target, so a human is the fixer, and a Stuck reason is what the operator sees on the envelope.
- **The merge commit stays.**
  On failure the task branch keeps the (possibly conflict-resolved) merge commit locally, and the operator fixes forward on top of it.
  Rationale: undoing it throws away a conflict session's work, and the operator's fix belongs on the merged tree.
- **Pending-verify marker.**
  After a tree-changing merge-in and before running the command, the producer writes a marker file at a told path;
  it removes the marker only when the command passes.
  If the marker exists when the producer reaches the gate, the gate runs even when this call's merge-in was a no-op.
  One marker shared by both producers; its content is not interpreted (presence is the signal).
  Rationale: without it, a resume after a Stuck verify finds the merge already done, skips the gate, and pushes the unfixed or unverified tree.
- **Output capture.**
  Combined stdout and stderr go to a told output file, overwritten on each run, and the Stuck reason names that path.
  Rationale: a Stuck reason must stay short, and the operator needs the compiler/test output to fix forward.
- **Both paths are told, never derived.**
  The marker and output file live under loom's scratch directory (never-tracked `.lyx` content), resolved by `internal/loomcli` through `loomengine`'s scratch accessors and told to landingshed alongside the verify command.
  Rationale: Told-Geometry and Durable-vs-Ephemeral invariants; landingshed may not name `.lyx` or compute a scratch path.
- **Execution: in-process, context-bound, in the task worktree root.**
  The command runs through the platform shell (`sh -c` on POSIX, `cmd /C` on Windows) with its working directory at `Deps.WorktreeRoot`, and is killed when the producer's context is cancelled.
  The shell-selection runner is shared with webster's `runVerifyCommand` (extracted to one place both call), not copied.
  A cancelled context returns the cancellation error (never Stuck) and leaves the marker in place.
  Rationale: one runner avoids two drifting shell-selection copies; context cancellation matches every other long step in the producers.
- **Placement inside `Finalize`'s retry loop.**
  The gate belongs to the merge-in step itself, so every merge-in `Finalize` runs — the first one and each retry after a merge-in-required refusal — is followed by the gate when it changed the tree.
  Rationale: the retry exists because the parent moved again, which is exactly a new unverified merge.
- **Unaffected paths.**
  The no-PR-required early return in `Publish`, the push-skipped refusal, and the operator-approval path (which syncs nothing) run no verify.

## Constraints

- **Told-Geometry Invariant:** landingshed stays free of `internal/lyxcwd` and derives no path; the verify command, marker path and output path are told through `Deps` by `internal/loomcli`.
- **Durable-vs-Ephemeral State Invariant / Lyxdirs Single-Declarer:** the marker and output file are never tracked, live under loom's scratch (`.lyx`) accessor, and no new file names the `.lyx` or `_lyx` literal.
- **landingshed's Fabric Vocabulary ban:** no identifier, string or comment in landingshed may name either fabric-internal side (enforced by `TestEnforcement_FabricVocabulary`).
- **`landingDeps` drift guard:** every new `Deps` field must be filled in `internal/loomcli/landingdeps.go`, and the every-field-populated guard must cover it.
- **Shed Recipe Registry / recipe rows:** no recipe row changes; `Publish` and `Finalize` keep no `on_stuck`.
- **Cancellation obligation:** every non-success exit consults `cancelErr` first, as `stuckOrCancelled` already does.
- The shared runner's new home must respect every leaf-package import allowlist in `CONSTRAINTS.md`; landingshed must not import `internal/websterengine`.
- Semantic line breaks in every Markdown file touched.

## Auto-mode assumptions

- Finalize is in scope although the board entry names only Publish (same bug class, same resolver, no reviewer downstream).
- The plan's `## verify:` is the only command source; a missing section is a warn-and-skip, not a halt and not a fallback.
- No verify timeout key: `go test`'s own per-binary timeout and context cancellation bound it in practice.
- No automated fix-forward session on failure; a human fixes.
- The marker is presence-only; no SHA is recorded in it.

## Open risks

- **Cross-package semantic breaks outside the plan's verify scope.**
  A plan whose `## verify:` names only the task's own packages misses a parent-side caller the task's signature change broke.
  The plan template permits `./...`, which closes it; this task does not force it.
- **Crash window before the marker is written.**
  A process killed between the merge-in completing and the marker write leaves an unverified merge with no marker, so a resume skips the gate.
  The window is one file write wide; accepted.
- **Verify cost on every tree-changing merge.**
  A plan-level suite that takes minutes now runs once more per landing whose parent moved, and again on each `Finalize` retry.
- **Output file size.**
  A very chatty suite writes a large scratch file; it is overwritten per run, never tracked.

## Acceptance criteria

- `Publish`, tree-changing merge-in, verify passes: push and PR create/refresh proceed as today; the marker is absent afterwards.
- `Publish`, tree-changing merge-in, verify fails: returns Stuck; no push, no GitHub client construction or call; the marker remains; the reason names the parent branch, the exit code and the output file path; the output file holds the command's output.
- `Publish`, merge-in already up to date, no marker: the verify command is not run.
- `Publish`, merge-in already up to date, marker present: the verify command runs; pass removes the marker and proceeds, fail is Stuck.
- `Publish`, empty verify command: no command runs, a warning is logged, push proceeds, no marker is written.
- `Publish`, spawn failure (the shell cannot start): Stuck with the spawn error in the reason.
- `Publish`, context cancelled while the command runs: the cancellation error is returned (not Stuck), and the marker remains.
- `Publish`, approval path and no-PR-required path: no verify runs.
- `Finalize`, tree-changing merge-in, verify fails: Stuck before the parent-side merge; the parent branch is unchanged, `MarkTaskDone` is not called, nothing is pushed.
- `Finalize`, tree-changing merge-in, verify passes: proceeds to the parent-side merge as today.
- `Finalize`, merge-in-required retry whose second merge-in changed the tree: the gate runs again after that merge-in.
- `internal/loomcli`: the verify command is resolved from the plan overview's `## verify:` at call time (a plan written after producer construction is picked up); the new `Deps` fields are populated and covered by the drift guard.
- webster's bisect and integration behaviour is unchanged after the runner is shared (its existing tests pass).
- The plan's own `## verify:` covers `internal/landingshed`, `internal/loomcli`, `internal/websterengine` and wherever the shared runner lands, including `-tags integration`.

## Notes for the plan writer

- The gate sits between `Publish.Call`'s step 4 (merge-in) and step 5 (push), and inside `Finalize`'s `mergeInStep`, which both its step 2 and its step 5 retry call.
- `mergeresolve.Result.AlreadyUpToDate` is the tree-changed signal; `OutcomeStuck` from the resolver still returns before the gate.
- Webster's runner is `runVerifyCommand` in `internal/websterengine/integration.go`; it takes no context and discards output today, so the shared form needs a context and an output writer.
- `planparser.Plan.Verify` already holds the chained command line; `planparser.PlanDir(location.AnchorPath())` is how loomcli reaches the plan elsewhere.
- landingshed's resolver seam is already faked in-package; a similar seam for the command runner keeps the unit tests off real processes.
