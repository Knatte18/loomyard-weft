## Goal

```markdown
Imported from [issue #45](https://github.com/Knatte18/loomyard/issues/45).

A pr-reviewer session runs under the publish deny (`shellguard` `SwitchDenyPublish`, `steerPublishDeny` in `internal/shellguard/rows.go`). That deny refuses `git push` and every `gh` call, including read-only ones such as `gh pr view`, `gh pr list` and `gh api` GET. lym also offers the agent no verb of its own to read a pull request.

As a result, a pr-reviewer cannot:
- look up which PRs merged into a branch
- read the PR's comments or review threads
- update the PR body after it rewrites `_lym/landing/summary.md`. PR-Review sets the body only when it enters the row. The summary also lives in the weft worktree, where the task worktree's git refuses the path, so the reviewer cannot commit it either.

On NORCE-DrillingAndWells/Models PR #265, #266 and #267, the orch had to relay each lookup and apply every body change with `gh pr edit`.

Wanted: the pr-reviewer may read and edit on GitHub, which covers `gh pr view`, `list`, `diff` and `checks`, `gh api` reads, `gh pr edit` for the title and body, and `gh pr comment` and `gh pr review` comments. `git push`, merging and closing stay with lym, so commits still pass the gate before they are pushed. As an alternative, PR-Review could resync the PR body from summary.md on every wake, not only on row entry.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - shellguard-quoted-gh-words-bypass-deny (class design, severity MEDIUM, open in rounds 1)
  - syncText-failed-edit-route-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
