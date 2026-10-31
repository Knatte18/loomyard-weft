# Decision record — teardown-retry-no-pair

## Goal

A batten run blocked at Worktree-Teardown after its pair was removed can be resumed with `lym batten run <slug>` on any machine that owns it, so the teardown retries only what is left, such as the remote branch deletion, and the run finishes.
The way forward the teardown's failed-deletion error names, and the one `lym batten cancel` names for a landed run, then works.

## Scope

- `lym batten run`'s resume pre-flight in `internal/battencli/owner.go`: a run whose status stands at Worktree-Teardown and whose task worktree is gone skips the run-scope origin reconcile.
- The local owner judgement that pre-flight falls back to, in the same file.
- The failed-deletion error of the Worktree-Teardown row in `internal/battencli/wire.go`, which names the resume verb.
- The battencli package doc in `internal/battencli/cli.go` and the comment on the resume path in `internal/battencli/arm.go`.
- Out of scope: `lym batten cancel`, which keeps refusing a landed run and naming `lym batten run`, now a way forward that works; `lym batten cleanup`; `fabricengine.ReconcileOrigin`, whose run scope keeps refusing a slug with no pair worktree.

## Decisions

### D1 — A teardown resume with no pair worktree skips the origin reconcile

The resume pre-flight skips the run-scope origin reconcile when the status file's current producer is Worktree-Teardown and the slug's task worktree is absent, and judges the owner against the local board instead.
Rationale: the reconcile refuses a run scope with no pair worktree, and a teardown whose pair is gone has no local branch of its own left to judge; the Worktree-Teardown row already finishes a half-removed pair and treats a pair of which nothing remains as done.
Bound: the skip covers only that producer with that worktree absent; every other resume still runs the reconcile.
What it lets through is a resume whose run origin has ended or stopped, judged only locally, and that resume can only run the teardown row, which removes what is left of the pair and deletes its remote branches.
An entry the local board shows owned by another machine is still refused.

### D2 — No origin-owed marker for the skipped reconcile

The skipped reconcile writes no origin-owed marker.
Rationale: the marker asks a later pass to reach origin for a reconcile this run can never run again once its pair is gone, and the only row left is the teardown, whose own pushes go to origin.

### D3 — The local owner refusal names a way forward that fits its cause

The local-board owner refusal keeps its re-run hint for an unreachable origin, and for a teardown resume with no pair names only `lym adopt <subject>` from the owning prime.
Rationale: a re-run once origin answers would skip the reconcile again, so naming it would be a way forward that does not help.

### D4 — The failed-deletion error names the verb that retries it

The Worktree-Teardown row's error for a remote branch left undeleted names `lym batten run <slug>` as the retry.
Rationale: the error is shown to the operator and dobby's overview, and D1 makes that verb work with the pair gone.

## Constraints

- `PATTERN-owner-machine`: a resume still runs only where the entry is not owned elsewhere, judged on the local board when the reconcile is skipped.
- `PATTERN-pair-teardown`: the teardown itself stays in `internal/pairteardown`, unchanged.
- `PATTERN-test-speed`: the new behavior is pinned by a tier0 table test over fake owner and reconcile seams.

## Auto-mode assumptions

- A darn run holds no interview; the board entry's Expected section is the whole spec.
- "On any machine" means any machine that owns the run; a machine that does not own it is still refused naming `lym adopt`.

## Open risks

- A run whose task worktree vanished by hand before Worktree-Teardown ever ran resumes without an origin judgement; the teardown row then removes what is left, which is the same outcome `lym fabric remove` gives such a pair.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - battenshed-doc-stale-on-resume-preflight (class consistency, severity MEDIUM, open in rounds 1)
  - pattern-owner-machine-stale-on-teardown-skip (class consistency, severity MEDIUM, open in rounds 1)
  - teardown-resume-pair-present-reconcile-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a resume at Worktree-Teardown with the task worktree absent runs no reconcile and resumes when the local board shows the run owned here or unowned.
- D1: a resume at Worktree-Teardown with the task worktree present still runs the reconcile, and so does a resume at any other producer.
- D1, D3: a resume at Worktree-Teardown with the task worktree absent and the local board showing another owner refuses naming that owner and `lym adopt`, without the "once origin answers" re-run.
- D2: the skipped reconcile leaves no origin-owed marker.
- D4: the failed-deletion error names `lym batten run <slug>`.
