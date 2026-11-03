---
round: 1
focus:
  - "internal/landingshed/prreview.go and prcontinue.go: does ending the review leave the pull request open, with the ending recorded on the row, and no remaining path closes it with `continueComment`?"
  - "internal/landingshed/finalize_pr.go: for a closed pull request whose record carries the ending, does Finalize reopen it and merge on GitHub; what happens when the reopen fails or the pull request was closed for another reason?"
  - "internal/landingshed/pullrequestmode.go and internal/pullrequest/client.go: does the branch-rules read (`GET repos/{owner}/{repo}/rules/branches/{branch}`) cover every row that would land by direct push, parse the `pull_request` rule correctly, and fail safe on an API error, a missing rule list or a branch without rules?"
  - "Finalize's parent side after a GitHub merge: do local and origin parent stay in step, and can any path still squash locally and push to a parent that requires a pull request?"
  - "Scope against the goal: does the diff change anything beyond the three expected behaviors (keep the pull request open, reopen when needed, rules-driven pull_request mode)?"
  - "PATTERN.md, stencil landing-template-pr-review.md, docs/overview.md and the package doc comments: do the edits match the new behavior, and does the stencil drop or weaken any rule the goal did not ask to change?"
  - "Test economy: do the new and changed tests in landingshed and pullrequest cover each added behavior (reopen, rules read, ending recorded) without duplicating an existing test, and is any added behavior uncovered?"
  - "The summary claims in `_lym/landing/summary.md` and the commit message: does the diff bear out each one?"
---

Seed focus from the goal in the decision record and the single-commit diff stat; the record's other sections are empty, so the goal's three Expected bullets are the measuring stick.
