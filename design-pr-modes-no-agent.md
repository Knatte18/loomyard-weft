# PR block in three modes, review, trail and off, with no agent under it

`pr-modes-no-agent` · Task · enhancement, landing, loom, config

Depends on: [`config-keys-nested`](design-config-keys-nested.md)

Note `pr-ask-live-pane-test` is folded in only to be closed: it asks for a test of the parent-review ask this task removes.

Bundled from the notes below; each part keeps its note's text.

# Take the parent out of PR-Review; the earlier reviewers must catch what it finds

Imported from [issue #223](https://github.com/Knatte18/loomyard/issues/223).

## Seen (operator, 2026-11-24)

Today the parent orch can be asked to review at PR-Review, and the operator has used that a lot.
The operator now judges it a bad idea, for two reasons.

PR-Review is the very last gate, and it belongs to the human.
When a run reaches it, everything the LLMs and LoomYard can do should already be done.

A parent is also the wrong reviewer.
Asking the parent, which is the orchestrator, to review a Discussion or code is work for the orchestrator, and it costs more than a cold-start agent.
The reviewers a run already has should catch what the parent catches.

## Decision (operator, 2026-11-26)

No agent reviews the pull request any more: when the PR is made, the agents are done.
What a parent finds when asked to review a PR must be found and fixed before the PR;
the checks that catch it are in the notes on reviewing the Describe text and on closing a task's scope at Code-Review.

## Idea

Take every agent out of PR-Review and add no seat in its place.

- Remove the parent's ask and re-ask from PR-Review in `internal/landingshed` and `internal/loomcli`.
- Remove PR-Review's own interactive session and its findings path: `pr_reviewer`, `lym loom pr-findings`, `pr_review_small_max`, `pr_review_ask_parent` and `pr_review_parent_timeout_min`.
- The row keeps only the mechanics the PR mode needs (see the note on the PR mode's three settings).
- Rely on the cold-start reviewers.
  The `review-code` fan already holds `code-scope`, which traces every decision of the record to a change in the diff and flags a decision no change builds, besides `code-correctness`, `code-affected-set` and `code-tests`.
  Read first which lenses the C# hubs run in `review-code` today, since the hub's own `lenses.yaml` and the C# layer can change the fan.
- Decide what a run does with a commit pushed to the PR branch by hand while it waits.


# A PR mode that registers the PR and approves it at once, with no agent under the block

Imported from [issue #224](https://github.com/Knatte18/loomyard/issues/224).

## Seen (operator, 2026-11-24)

The operator often wants a pull request registered on GitHub without looking at it.
With a beam, say P1, and several tasks under it, the operator wants each task to land as a PR that is approved at once and merged into the beam.
When all of P1 is done, one PR goes from P1 into its parent (P0 or root), and it lists every member PR.
That lets the operator backtrack and see how the work got where it is.

## Decision (operator, 2026-11-26)

The PR block has three modes, and no LLM takes part in any of them:

- Manual: the PR is made and waits for the operator's own review and approval, as PR-Review is meant today.
- None: no PR; the row passes, as `pull_request` off does today.
- Trail (`pull_request: trail`, operator 2026-11-26): the PR is made and merged at once, with no agent and no human, so the beam's PR history can be backtracked.

How the three modes are configured is not decided yet.

## Idea

- Trail: the PR is made, no agent is spawned, and Finalize merges it at once, as it does today after the operator's `lym loom pr-continue`; merging a PR without an approval has long worked in lym.
- A beam's own PR lists its member PRs.
  `Rewrite a beam's pull request description after each member landing` (`1cd11ee5e`) may already do part of this; read what it writes before adding more.
- Manual waits for a human approval whatever the autonomy, which covers the note on autonomy and PR-Review.
- Read first where `beam.yaml` `pull_request` is read (`internal/landingshed`, `internal/pullrequest`) and how a run reaches Finalize today when a PR is approved.


# Autonomy: let PR review stay with the operator for one beam

Imported from [issue #217](https://github.com/Knatte18/loomyard/issues/217).

`beam.yaml: autonomy` is one switch for both halts and PR-Review requests. A hub wants the orch to settle halts on the prime autonomously while pull requests into the prime's branch (main) still wait for the operator's approval. Today that needs autonomy off, which also sends every halt to the operator, or autonomy on with the orch holding PR-Review by hand. Proposal: a separate key, such as `autonomy_pr_review: false`, so PR-Review requests on that beam or the prime go to the operator while halts stay autonomous.

## Issue comment, 2026-11-24

Seen again, with a merge into the prime's branch:

- The prime runs with `autonomy: true`, and the hub wants pull requests into main to wait for a human approval.
- The task `import-project-names` reached PR-Review; the parent answered the parent-review ask `--satisfied` and told the PR-Review agent by creel that landing waits on the operator's approval.
- `lym loom pr-continue` ran anyway about a minute later, and Finalize squash-merged the pull request into main once its gate passed, with no human approval on GitHub.

A message to the PR-Review agent is no way to hold a landing under autonomy: the satisfied answer alone ends the review.

Expected: a setting, per beam and for the prime, that keeps autonomy for halts but makes PR-Review wait for a human approval of the pull request (for example a GitHub approving review) before `pr-continue` or Finalize may merge.

# No tmux-tier test sends parent-review ask from a live non-orch pane

## Seen

`failed-ask-hides-satisfied` (issue 213) left out its item 3: no test runs `lym loom parent-review ask` from a live non-orch reed strand pane.
That is the path that failed twice on 2026-11-23 in the mo hub (creel refused the LOOM signature), and the fix (708cdadc5) is pinned only below the tmux tier.

## Idea

A tmux-tier test: a built lym binary in a reed strand pane, a hub with a pull request record under autonomy, and a live orch strand that receives the post; assert the ask is delivered and the parent's answer lets `pr-continue` end the review.



# README display

The board README shows each beam's pull-request setting (task `readme-beam-modes`); show the three modes there.
