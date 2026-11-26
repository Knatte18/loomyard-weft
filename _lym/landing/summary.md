# PR modes review, trail and off, with no agent at PR-Review

`beam.pull_request` now names the three PR modes, and no agent takes part in any of them.

- **`review`** (was `on`): PR-Review opens the pull request and waits for the operator's own ending, whatever `beam.autonomy` says.
- **`trail`** (was `merge`): Finalize opens the pull request and squash-merges it at once, with no review, so a beam's history keeps a pull request per member to backtrack through.
- **`off`**: no pull request, unless GitHub's rules for the parent branch require one.

`merge-in` is unchanged.

The PR-Review row's interactive session is gone, and so are the parent orch's review ask and the send-back of findings to Code-Review.
When the pull request is made, the run's agents are done.
The cold-start reviewers have to catch what a parent review caught.
The `review-code` fan already runs `code-scope`, `code-correctness`, `code-affected-set` and `code-tests`, in the shipped lenses and in the C# set, which adds `numerics`.

## What changes

### The modes (`internal/beamsetting`, `internal/dobbyengine`, `internal/landingshed`, `internal/beamcli`)

- `beamsetting.PullRequestReview` and `PullRequestTrail` replace `PullRequestOn` and `PullRequestMerge`.
  `ResolvePullRequest` keeps its logic under the new names.
  The beam template's default is `review`.
- A new reinterpretation, `beam-pull-request-modes`, rewrites the hub's `beam.yaml`: `on` becomes `review` and `merge` becomes `trail`, each keeping what it meant.
  It flags each worktree's own `beam.yaml` that still holds an old value and names the `lym config beam --set` that fixes it.
  `CarryBeamTaskSettings` carries a board entry's old value as the new one.
  The older `beam-per-task-settings` reinterpretation now maps `required` to `review`.
- The board README's mode legend states what each mode does.

### PR-Review (`internal/landingshed/prreview.go`)

- Under `review` the row pushes, opens or finds the pull request, records it and waits.
  The wait ends on a merge or an approval on GitHub, or on `lym loom pr-continue`.
- A commit made in the task worktree while the row waits is still gated and pushed.
  A gate failure now halts the row, since no session is left to mail.
  The halt names the way forward, and the resume gates the task head again.
- A commit pushed to the pull request's branch on GitHub by hand is a commit the task worktree lacks, and it halts the row.
  The halt says to bring the commit into the task branch, or drop it from GitHub, then resume.
  A GitHub head that only lags behind a pushed head keeps the wait going, and the check uses `CommitsMissing` for this.
- Removed: `prsession.go`, `prreviewask.go`, the `PullRequestDeps` seams the session and the ask needed, and Finalize's recorded-autonomy check on a closed pull request.

### Verbs (`internal/loomcli`)

- `lym loom pr-continue` is now an operator verb.
  It refuses any call from a lym agent session (`LYM_STRAND_NAME` set), the orch's included, with a way forward that tells the operator to end the review.
  It no longer reads a parent answer or clears a mailed head.
- Removed: `lym loom pr-findings`, `lym loom parent-review ask`, the goto hook that cleared the PR-Review session, and the `pr-findings-<n>.md` review input.
  `parent-review answer` and `await` stay for the review rounds and the drift judge.
  `burlerengine.AnswerParentAskOrUnreached` becomes `AnswerParentAsk`, with no unreached-tag exception.
- `shedadapters.ReviewInput.Full` is gone, since a PR-findings file was the only input that forced a full generation.

### Config, records and contracts

- Retired keys: `landing.pr_reviewer`, and the `loom.pr` area: `loom.pr.review_ask_parent`, `loom.pr.review_parent_timeout_min` and `loom.pr.review_small_max`, with their old flat names.
  The `agents.yaml` role `pr-reviewer` is gone.
  Autonomy no longer forces a PR-Review ask.
- `pullrequest.Record` drops its session, autonomy, ask, send-back and mailed-head fields.
  An older record's keys survive a rewrite and are never read.
- The three PR-Review stencils are deleted.
  The darn, sampler and tapestry recipes drop PR-Review's `on_reroute: Code-Bouncer`.
- `orch-role.md` and `role-autonomy.md` say a run waiting at PR-Review is left to the operator.
  `PATTERN-landing.md`, `docs/overview.md`, the code-review rubric and the final-summary spec are updated.

## Design decisions

- **Mode names.** The board entry left the configuration open.
  The task title names `review`, `trail` and `off`, and the operator chose `pull_request: trail`.
  The old names move through a reinterpretation, as `PATTERN-config-layers` asks of a meaning change.
- **The operator's ending.** GitHub does not let the operator approve a pull request lym opened with the operator's own token.
  So `pr-continue` stays as the operator's ending, and Finalize merges after it, as before.
  The verb itself refuses an agent session, as `lym config` already does for a session with a slug, so no shell-guard row is added.
  It is a guardrail against a mistaken call, not a barrier.
- **Hand-pushed commits** halt rather than being fetched and gated, because the run never gated them and the operator who pushed them decides.
- **A gate failure halts.** It used to be mailed to the session's agent; now the operator, or the orch under autonomy, settles it.
- **Beam pull requests** already list their member pull requests: since `1cd11ee5e`, the beam describer rewrites the beam's description after each member landing, naming every member pull request by number.
  No further change was needed.
- **`pr-ask-live-pane-test`** asked for a test of the parent-review ask, which this change removes, so the note closes with no test.

## Merge danger

**Door:** one-way
**Blast radius:** landing
