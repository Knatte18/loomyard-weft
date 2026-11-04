```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: behaviour → pinning test**

Shellguard (`internal/shellguard/rows_integration_test.go`, `TestRows_PatternsDenyAndAllow`, lines 226–260):

| Behaviour | Pinning row |
|---|---|
| `git push` refused | `git_push`, `git_push_after_dash_c` (existing) |
| Unlisted gh command refused | `unlisted_gh_command`, `unlisted_gh_auth_token`, `unlisted_gh_command_sharing_a_prefix` |
| Unlisted `pr` subcommand refused (merge, close, create, update-branch) | `gh_pr_merge`, `gh_pr_close`, `gh_pr_create`, `gh_pr_update_branch` |
| Unlisted subcommand refused after a separator | `gh_pr_merge_after_a_separator` |
| `-R` between command and subcommand | deny `gh_pr_merge_after_a_repo_flag`, allow `gh_pr_view_after_a_repo_flag` |
| `pr review --approve` refused | `gh_pr_review_approve` |
| `pr review -r` refused | `gh_pr_review_request_changes` |
| `pr edit --base` refused | `gh_pr_edit_base` |
| `api` write method refused | `gh_api_post`, `gh_api_lowercase_method` |
| `api --input` refused | `gh_api_input` |
| `api` with `mutation` refused | `gh_api_graphql_mutation` |
| Dash-opening word passes | `gh_version` |
| Listed reads, comments and title/body edits pass | `gh_pr_view`, `gh_pr_list`, `gh_pr_diff`, `gh_pr_checks`, `gh_pr_edit_title_and_body`, `gh_pr_comment`, `gh_pr_review_comment`, `gh_api_read`, `gh_api_get_with_fields`, `gh_api_graphql_query`, `gh_run_view`, `gh_search`, `gh_pr_view_piped` |
| Description field not read as a command | `publish` description row `x; gh pr merge 3` |
| Steer and notice carry no forbidden characters | `init()` / `checkTexts` panic at package load |

Landingshed (`internal/landingshed/prreview_wait_test.go`, `TestPRReview_Wait`):

| Behaviour | Pinning row |
|---|---|
| Edit after a changed description, at the next poll, once | `AnEditedDescriptionSetsTheTitleAndBodyAgainAtTheNextPollAndOnlyThen` |
| Unparsable description: one warning, no edit | `ADescriptionThatYieldsNoTextIsLoggedOnceAndEditsNothing` |
| No resync once the continue flag is set | `AnEditedDescriptionIsNotSetAfterTheOperatorsEnding` |
| `pullRequestText` refactor at entry | Existing `prreview_test.go` entry tests. The error wrapping is unchanged. |

**Findings**

- **F-T1, MEDIUM, design.** The failed-edit route has no test row, at `internal/landingshed/prreview.go:346-348`.
  - The failed edit in `syncText` goes through `pollFailure`. The summary claims it is "judged the same way as a failed poll".
  - No row makes `Edit` fail, so these are unpinned: a transient failure warning once and continuing, a non-transient failure ending Stuck, and `state.synced` staying old so the next poll retries.
  - Fix: add a `TestPRReview_Wait` row where `Edit` returns a transient error, then succeeds, and one where it returns a non-transient error.

- **F-T2, LOW, design.** Two of the three resync gating conditions have no row, at `prreview.go:552`.
  - Only `rec.Continue` is tested. `status.State == StateOpen` and `!rec.ReviewEnded` are not.
  - Dropping either condition would pass every test, so a merged or closed PR, or an ended review, could be edited.
  - Fix: add rows where the status is merged or closed, and where the record has `ReviewEnded`, each with a rewritten description and no `Edit` call expected.

- **F-T3, LOW, design.** The zero-baseline branch has no row, at `prreview.go:340-342`.
  - `state.synced == prText{}` takes the present text as the baseline and edits nothing. This is reached when `wait` gets `prText{}` and later sees the PR open.
  - No test covers it. The existing row reaches `wait` with text already set.
  - Fix: add a row that enters the wait with a zero `synced`, an open status and a changed description, and expects no `Edit` at the first poll.

- **F-T4, LOW, design.** The warning re-arm has no row, at `prreview.go:338`.
  - The no-text row proves one warning over two polls. It does not prove `textWarned` resets, so a second episode would warn again.
  - Fix: extend that row, or add one, with a bad description, then a good one, then a bad one, expecting two warnings.

- **F-T5, LOW, scope.** Several shellguard branches are unpinned, at `rows.go:296-300` and `rows.go:200-208`.
  - `pr review -a` and `--request-changes`.
  - `pr edit -B`.
  - `api` with `PATCH`, `DELETE` or `-XPOST` without a space.
  - `--repo=` and `-R` placed before the command (`gh -R o/r pr merge`).
  - `--input` is covered, but a `mutation` on a later line, which the `\n` exclusion lets through, is not.
  - Dropping any of these alternatives passes every test.
  - Listed subcommands of `issue`, `repo` and `run` are never denied (for example `gh issue create`, `gh repo delete`, `gh run rerun`). `gh run watch` and `gh status` are never allowed. Only the `pr` subcommand trie is exercised, so a typo in the table data would go unnoticed.
  - Fix: add compact deny rows for `gh pr review 3 --request-changes`, `gh pr edit 3 -B x`, `gh api --method=DELETE x`, `gh --repo o/r pr merge 3` and `gh issue create`, and allow rows for `gh run watch 5` and `gh status`. Put them in the same table.

**Not findings**
- The last commit (`c649a96a`) only adds the eighth wake's status poll to the operator-ending description test, so the test's meaning is unchanged.
- No new row duplicates an existing one.
- The old `gh_after_a_separator` deny row (`gh api y`) was removed because `gh api` is now allowed. Its separator coverage moved to `gh_pr_merge_after_a_separator`.

**Tool calls (1 of 6)**
1. `grep` over test files in `shellguard`, `shuttleengine` and `samplerengine` for publish-row steer text assertions. My context lacked whether any test asserts the old "git push and gh are refused" wording. None does, so no stale assertion exists.
