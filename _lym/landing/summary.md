# PR modes review, trail and off, with no agent at PR-Review

`beam.pull_request` now names the three PR modes, and no agent takes part in any of them.
PR-Review is the last gate and belongs to the operator:
when the pull request is made, the run's agents are done, and the earlier cold-start reviewers have to catch the rest.

- **`review`** (was `on`): PR-Review opens the pull request and waits for the operator's ending,
  whatever `beam.autonomy` says.
- **`trail`** (was `merge`): Finalize opens the pull request and squash-merges it at once, with no review,
  so the parent branch keeps a pull request per member to backtrack through.
- **`off`**: no pull request, unless GitHub's rules for the parent branch require one.

`merge-in` is unchanged.

Closes #223
Closes #224
Closes #217

## What changes

### Modes and the upgrade of old values

- `beamsetting.PullRequestReview` and `PullRequestTrail` replace `PullRequestOn` and `PullRequestMerge`.
  The beam template's default is `review`.
- The reinterpretation `beam-pull-request-modes` rewrites `on` and `merge` in the hub's `beam.yaml`,
  and flags a worktree's own `beam.yaml` that still holds one.
  `profiles-pull-request-modes` does the rewrite in the hub's `profiles.yaml`.
- The reconcile walk renames old values in board entries' `run_settings`, and `CarryBeamTaskSettings` carries them renamed.
  The older `beam-per-task-settings` reinterpretation now maps `required` to `review`.
- The board README's mode legend states what each mode does.

### PR-Review

- Under `review` the row pushes, opens or finds the pull request, records it and waits.
  The wait ends on a merge or an approval on GitHub, or on `lym loom pr-continue`.
- A commit made in the task worktree during the wait is gated and pushed.
  A gate failure halts the row, and the resume gates the task head again before pushing it.
- A commit pushed to the pull request's branch on GitHub by hand, which the task worktree lacks, halts a review that has not ended.
  The way forward is to bring the commit into the task branch or drop it from GitHub, then resume.
  Not every push path checks for it; see "To check" below.
- The row's interactive session, the parent's review ask and the send-back of findings to Code-Review are removed.
  So is Finalize's recorded-autonomy check on a closed pull request.

### Verbs, config and contracts

- `lym loom pr-continue` is now an operator verb and refuses any lym agent session, the orch's included.
- `lym loom pr-findings`, `lym loom parent-review ask`, the goto hook that cleared the PR-Review session
  and the `pr-findings-<n>.md` review input are removed.
  `parent-review answer` and `await` stay.
- Retired keys: `landing.pr_reviewer`, `loom.pr.review_ask_parent`, `loom.pr.review_parent_timeout_min`
  and `loom.pr.review_small_max`, with their old flat names.
- The `agents.yaml` role `pr-reviewer` is gone.
  The reinterpretation `agents-pr-reviewer-role-retired` removes it from the hub's file and flags a local one.
- `pullrequest.Record` drops its session, autonomy, ask, send-back and mailed-head fields.
  An older record's keys survive a rewrite and are never read.
- The PR-Review stencils are deleted, and the darn, sampler and tapestry recipes drop PR-Review's `on_reroute`.
- The orch role files, `PATTERN-landing.md`, `PATTERN-config-layers.md`, `docs/overview.md`,
  the code-review rubric and the final-summary spec follow.

## Design decisions

- **The operator's ending.** GitHub does not let an author approve their own pull request,
  and lym opens it with the operator's token, so `pr-continue` stays as the operator's ending.
  Its refusal of agent sessions reads the strand name in the environment:
  a guardrail against a mistaken call, not a barrier.
- **Hand-pushed commits** halt rather than being fetched and gated, since the run never gated them.
- **A gate failure halts**, as no session is left to mail.
- **Beam pull requests** need no change to list their members: the beam describer already names every member pull request.
- The note `pr-ask-live-pane-test` asked for a test of the parent-review ask, which this removes, so it closes with no test.

## Upgrading

- Every machine and pair of a hub needs the new binary before the reinterpretation rewrites the hub's `beam.yaml`,
  since an older binary refuses `review` and `trail`.
- Local `beam.yaml` and `agents.yaml` files are flagged only on the machine whose walk applies the reinterpretation.
- A run waiting at PR-Review during the upgrade keeps its PR-Review session, whose prompt names removed or refused verbs.
  Stop it with `lym reed remove <strand_guid>` in the task worktree;
  the guid is the `strand_guid` key of the run's `pull-request.json`.

## To check

- **A push can still drop a hand commit made on GitHub.**
  Decide before landing whether this is acceptable.
  For a task pair, a rejected push is retried by `resetUnnamedTip` (`internal/fabricengine/pushclassified.go`),
  which can force-reset origin's code branch past the hand commit.
  This is read from the code and was not tested against a remote.
- The finding is that PR-Review pushed before it checked GitHub's head for a hand commit (`internal/landingshed/prreview.go`).
  The wait and the row's entry on an open, unended review now check first.
  These paths can still push over the commit:
  - a re-entry on an ended review (`openPullRequest`);
  - a re-entry on a pull request GitHub reports closed, at the gated head (`openPullRequest`);
  - `gateHead`'s push after a long gate, since the status is read only before the gate;
  - Finalize's push before it merges (`mergeOnGitHub`, `internal/landingshed/finalize_pr.go`);
  - the run's background pusher.

## Merge danger

**Door:** one-way
**Blast radius:** hub
