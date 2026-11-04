# Let the pr-reviewer read and edit on GitHub

The PR-Review session ran under a publish shell guard that refused `git push` and every `gh` call, read-only ones included.
So the pr-reviewer could not look up merged PRs, read a PR's comments or review threads, or carry a rewritten `_lym/landing/summary.md` onto the PR body, and the orch had to relay each lookup and body change by hand.
This change narrows the guard to what has to stay with lym, and has PR-Review keep the PR's title and body in step with the change description.

## The publish guard becomes an allowlist

The `publish` row in `internal/shellguard/rows.go` still refuses `git push`.
For `gh` it now refuses every command and subcommand that the new `githubAllowed` table does not list.
The table lists `gh pr view`, `list`, `diff`, `checks`, `status`, `edit`, `comment` and `review`, `gh run` and `gh issue` reads, `gh repo view`, `gh search`, `gh status` and `gh api`.
Merges, closes, reopens, PR creation, `gh pr update-branch`, releases and `gh auth` stay refused.
Some calls are refused even though their command is listed:

- `gh pr review` with `--approve` or `--request-changes`.
- `gh pr edit` with `--base`.
- `gh api` with a writing `-X`/`--method`, with `--input`, or with `mutation` anywhere in its command.

ERE has no negative lookahead, so `notWordPattern` builds a trie over the listed words and generates a pattern that matches any word except them.
A gh subcommand added later is therefore refused until the table lists it.
The steer and notice text is generated from the same table, so the allowed calls are named in one place.
`-R`/`--repo` between a command and its subcommand is read as a flag, so `gh pr -R o/r merge 3` is still refused.

## PR-Review resyncs the title and body

Each status poll in the PR-Review wait (`internal/landingshed/prreview.go`) re-parses the change description, but only when the poll finds the PR open with no continue flag or ending recorded.
When the description yields different text than the PR was last set to, PR-Review calls `Edit` through the existing GitHub client.
That text is the title, plus the body with the regression check and credit block appended.
A description that does not parse, for example one caught mid-edit, logs one Warn per episode and is retried at the next poll.
A failed edit is judged the same way as a failed poll.
The entry's own title-and-body computation moved into `pullRequestText`, so the entry and the resync build identical text.

The pr-reviewer stencil now names the `gh` calls it may make.
It tells the agent to change the title or body by editing the change description, not with `gh pr edit --body`, which would drop the regression check and credit block.

## Design decisions

- **Allowlist rather than a denylist of publishing calls.** gh grows subcommands that push, such as `pr update-branch`, so refusing unknown words fails safe.
- **Both halves of the entry, not one.** The entry offered the resync as an alternative to `gh pr edit`. Opening `gh` alone would have made the agent assemble the regression check and credit block itself. The resync keeps the PR body exactly what Finalize's landing message uses.
- **The sampler shares the switch.** `samplerengine` also sets `DenyPublish`, so a sampler session gets the same gh reads and comments. Nothing it does needs a stricter guard, and a second switch would add a row with no caller.
- **`gh api` field flags pass.** A GraphQL query and a GET with query parameters both use `-f`, so the row cannot refuse an implied POST without refusing those. The row comment names this hole next to the existing guardrail list.

## Testing

- `TestRows_PatternsDenyAndAllow` gains deny rows for the refused calls above. It also gains allow rows for every listed read, comment and edit form, a `-R` flag, a pipe, `gh --version`, and a GraphQL query. The description-field case for `publish` was added too.
- `TestPRReview_Wait` gains three rows: an edited description is set at the next poll and not again; a description that yields no text warns once over two polls and edits nothing; and no resync happens once the continue flag is recorded.
