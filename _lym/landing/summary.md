# Let the pr-reviewer read and edit on GitHub

A pr-reviewer session ran under a publish deny that refused `git push` and every `gh` call, so it could not look up merged PRs, read a PR's comments or review threads, or change the PR body.
The orch had to relay each lookup and apply each body change by hand.

The pr-reviewer now reads the pull request on GitHub, comments on it, and changes its title and body by rewriting the change description.
Pushing, approving, merging and closing stay with lym.

Closes #45

## What changes

- `shellguard`: the publish deny still refuses `git push`, and now lets through a fixed set of `gh` calls: `gh api`, `gh pr checks|comment|diff|edit|list|review|view` and `gh run view`.
  Within that set it still refuses `gh api` with a writing `-X`/`--method`, with `--input`, or with a GraphQL `mutation`; `gh pr review` with an approve or request-changes flag; and `gh pr edit` with a new base.
  The allowlist lives in `githubAllowed`, and the pattern and the steer text are both built from it.
- `landingshed` PR-Review: the wait re-reads the change description at each status poll of an open pull request, and sets the title and body again when the text differs from what was last set.
  This is the issue's alternative, chosen over letting the reviewer run `gh pr edit` for the body, because the body must carry the regression check and the credit block after the description's own text, and `gh pr edit` would drop them.
  A text that cannot be built, such as a description caught mid-edit, is logged once and retried at the next poll.
  A failed edit follows the same transient or hard failure judgment as a failed status read.
- The PR-Review stencil tells the reviewer which `gh` reads and comments it has, and to change the title or body by editing the description, never with `gh pr edit`.
- `contracts/specs/loom-final-summary-spec.md` records the re-read.

## Design decisions

- The deny is a guardrail, not a barrier, as before; the row's comment lists the known gaps.
- A review ends the sync: it stops once the review has ended or the operator continued, so the reviewer's edit after that point is not pushed to the PR.

## To check by hand

- `shellguard`, quoted `gh` words: a command word or a method word in quotes is handled by the pattern, but the review left open that a quoted spelling might still get past the deny. Try a few quoted variants of `gh pr merge` and `gh api -X POST` against the row.
- `landingshed` `syncText`: the path where the title and body edit fails is not covered by a test. Check it by hand, or add a case to `prreview_wait_test.go`.
- No integration run against a real pull request was done; try one description edit in a live PR-Review and confirm the title and body update within a minute.
