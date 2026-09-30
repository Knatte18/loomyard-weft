# Publish: verify the tree after merging the parent

## Goal

Stop the landing producers from pushing or landing a tree that a parent merge-in broke, within what the plan's `## verify:` command covers.
Today `Publish` merges the parent into the task branch through `internal/mergeresolve` and pushes without compiling or testing the result;
a merge can compile cleanly and still break tests, as a conflict-resolved merge in run `operator-surface` did.
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
  A stuck resolver outcome (the merge was aborted) still returns before the gate, as today: no verify runs and no marker is written.
  Rationale: a clean textual merge can still break compilation (a semantic conflict); only a no-op merge leaves the tree webster already verified.
- **Both landing producers carry the gate.**
  Rationale: `Finalize`'s merge-in has the identical failure mode and is worse — it lands on the parent and pushes with no reviewer downstream, both on no-PR (task-to-task) landings and when the parent moved while an approved PR sat in review.
- **Command source: the plan's `## verify:` section, read when the gate runs.**
  `Deps` carries a closure, filled by `internal/loomcli`, that parses the plan through `internal/planparser` each time the gate runs and returns its already-chained single command line (the one webster runs);
  landingshed never reads or names the plan.
  It is a closure, not a value: `landingDeps` runs at bootstrap, before the plan exists on a fresh run, so a string filled there would be empty and the gate would silently skip.
  A plan read or parse error is a returned producer error, never a skip.
  Rationale: the plan's verify is repo-agnostic, and the plan template plus plan-review rubric already require it to cover every targeted package including build-tagged tests.
- **No command means no gate.**
  Only a parsed plan with no `## verify:` section yields an empty command.
  An empty command skips the gate with a `logger.Warn` naming the skip, whether or not a marker exists: nothing runs, and no marker is written or removed;
  the warning names the pending marker when one is present.
  The producer then proceeds as today.
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
  A failure to write the marker or to create the output file is a returned error, never Stuck, matching a `CommitStatus` failure.
  Rationale: without it, a resume after a Stuck verify finds the merge already done, skips the gate, and pushes the unfixed or unverified tree;
  a scratch-file write failure is infrastructure, not a tree a human fixes.
- **Output capture.**
  Combined stdout and stderr go to a told output file, overwritten on each run, and the Stuck reason names that path.
  Rationale: a Stuck reason must stay short, and the operator needs the compiler/test output to fix forward.
- **Both paths are told, never derived.**
  The marker and output file live under loom's scratch directory (never-tracked `.lyx` content), resolved by `internal/loomcli` through `loomengine`'s scratch accessors and told to landingshed alongside the verify command.
  Rationale: Told-Geometry and Durable-vs-Ephemeral invariants; landingshed may not name `.lyx` or compute a scratch path.
- **Execution: in-process, context-bound, in the task worktree root.**
  The command runs through the platform shell (`sh -c` on POSIX, `cmd /C` on Windows) with its working directory at `Deps.WorktreeRoot`.
  A cancelled context kills the shell process, and a bounded wait delay keeps the call from hanging on output after the kill, as `internal/treadleengine`'s gate command already does.
  A cancelled context returns the cancellation error (never Stuck) and leaves the marker in place.
  Rationale: context cancellation matches every other long step in the producers.
- **One shared runner, webster's behaviour unchanged.**
  The shell-selection runner is shared with webster's `runVerifyCommand` (extracted to one place both call), not copied, and gains a context and an output writer.
  Webster passes a background context and a discarding writer, so its integration-failure bisect keeps today's behaviour;
  the shared runner keeps today's spawn and teardown log lines for both callers.
  Rationale: one runner avoids two drifting shell-selection copies.
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
- Every Decision above is a self-pick; the support log's Question ledger records each one.

## Open risks

- **Cross-package semantic breaks outside the plan's verify scope.**
  A plan whose `## verify:` names only the task's own packages misses a parent-side caller the task's signature change broke.
  The plan template permits `./...`, which closes it; this task does not force it.
- **Crash window before the marker is written.**
  A process killed between the merge-in completing and the marker write leaves an unverified merge with no marker, so a resume skips the gate.
  The window is one file write wide; accepted.
- **Orphaned children of a cancelled verify.**
  Cancellation kills the shell, not the commands a chained verify line started under it;
  those may run on and write to the output file after the producer returns.
- **Verify cost on every tree-changing merge.**
  A plan-level suite that takes minutes now runs once more per landing whose parent moved, and again on each `Finalize` retry.

## Acceptance criteria

- `Publish`, tree-changing merge-in, verify passes: push and PR create/refresh proceed as today; the marker is absent afterwards.
- `Publish`, tree-changing merge-in, verify fails: returns Stuck; no push, no GitHub client construction or call; the marker remains; the reason names the parent branch, the exit code and the output file path; the output file holds the command's output.
- `Publish`, merge-in already up to date, no marker: the verify command is not run.
- `Publish`, merge-in already up to date, marker present: the verify command runs; pass removes the marker and proceeds, fail is Stuck.
- `Publish`, resolver returns a stuck outcome: Stuck as today; no verify runs and no marker is written.
- `Publish`, empty verify command: no command runs, a warning is logged, push proceeds, no marker is written.
- `Publish`, empty verify command with a marker present: no command runs, the warning names the pending marker, the marker stays, push proceeds.
- `Publish`, plan read or parse error: a returned error, not Stuck; no push.
- `Publish`, marker write or output-file create failure: a returned error, not Stuck; no push.
- `Publish`, spawn failure (the shell cannot start): Stuck with the spawn error in the reason; the marker remains.
- `Publish`, context cancelled while the command runs: the cancellation error is returned (not Stuck), and the marker remains.
- `Publish`, approval path and no-PR-required path: no verify runs.
- `Finalize`, tree-changing merge-in, verify fails: Stuck before the parent-side merge; the parent branch is unchanged, `MarkTaskDone` is not called, nothing is pushed.
- `Finalize`, tree-changing merge-in, verify passes: proceeds to the parent-side merge as today.
- `Finalize`, merge-in already up to date, marker present: the verify command runs; fail is Stuck before the parent-side merge, pass removes the marker and proceeds.
- `Finalize`, merge-in-required retry whose second merge-in changed the tree: the gate runs again after that merge-in.
- `internal/loomcli`: the verify command is resolved from the plan overview's `## verify:` at call time (a plan written after producer construction is picked up); the new `Deps` fields are populated and covered by the drift guard.
- webster's integration-failure bisect is unchanged after the runner is shared: it passes a background context and a discarding writer, and its existing tests, including the spawn and teardown log assertions, pass.
- The plan's own `## verify:` covers `internal/landingshed`, `internal/loomcli`, `internal/websterengine` and wherever the shared runner lands, including `-tags integration`.

## Notes for the plan writer

- The gate sits between `Publish.Call`'s step 4 (merge-in) and step 5 (push), and inside `Finalize`'s `mergeInStep`, which both its step 2 and its step 5 retry call.
- `mergeresolve.Result.AlreadyUpToDate` is the tree-changed signal; `OutcomeStuck` from the resolver still returns before the gate.
- Webster's runner is `runVerifyCommand` in `internal/websterengine/integration.go`; it takes no context and discards output today, so the shared form needs a context and an output writer.
- `planparser.Plan.Verify` already holds the chained command line; `planparser.PlanDir(location.AnchorPath())` is how loomcli reaches the plan elsewhere.
- landingshed's resolver seam is already faked in-package; a similar seam for the command runner keeps the unit tests off real processes.
