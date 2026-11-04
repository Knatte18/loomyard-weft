---
round: 1
focus:
  - "internal/shellguard/rows.go `publishCommandPattern` and `notWordFrom`: settle whether the generated grep -E allowlist refuses what the entry says stays with lym (git push, merge, close, reopen, create, update-branch, auth, release) and lets through every read, comment and title/body edit it names; probe flag-word, `-R`/`--repo`, case and quoting variants of `gh pr merge`, `gh pr close`, `gh pr review --approve` and `gh api -X POST` / `--input` / `mutation`."
  - "internal/shellguard/rows.go `githubAllowed`: settle whether any listed command or subcommand (`gh run watch`, `gh api` with any method, `gh search`, `gh status`, `gh issue`) goes beyond the entry's wanted set (`pr view/list/diff/checks`, `api` reads, `pr edit` title/body, `pr comment`, `pr review` comments), and whether `gh pr edit` can still change anything the entry says stays with lym."
  - "internal/landingshed/prreview.go `syncText`, `wake` and `wait`: settle whether the resync edits only while the PR is open and the review is going, whether `prText{}` as the zero value for an unknown text and the `synced` hand-off from `openPullRequest` to `wait` leave a path that skips an edit or edits twice, and whether a failed edit takes the same route as a failed poll."
  - "internal/landingshed/prreview.go `pullRequestText` against the removed inline code in `openPullRequest`: settle whether the entry path builds the same title and body as before, and whether error wrapping and messages stayed equivalent."
  - "The three `wait`/`openPullRequest` call sites that pass `prText{}` (operator-ended head match, the already-open record path): settle whether skipping the first-poll sync there is intended and consistent with the doc comments."
  - "Tests: settle whether the new `TestPRReview_Wait` rows and the new deny/allow rows in `rows_integration_test.go` cover every behavior the diff adds without duplicating existing rows, and whether the eighth-wake status-poll fix in the last commit changes an existing test's meaning."
  - "`_lym/landing/summary.md` against the diff: settle whether each claim holds, including the listed allowed commands, the three Wait rows, the description-field case for `publish`, and the stencil wording that tells the agent to edit the change description rather than use `gh pr edit --body`."
  - "contracts/specs/loom-final-summary-spec.md, landing-template-pr-review.md and the doc.go edits: settle whether the prose matches the code as it now behaves and carries what the entry asks the stencil to name."
---

The record holds only the imported entry under `## Goal`, so the entry is the measuring stick.
Focus favors the guard's pattern generation, which has the widest failure surface, and the resync's state handling.
