```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1 through F6) by building cases where a proposed change goes wrong.

no findings

I built and tested these cases, and none broke a proposal:
- **F1:** I tried four cases.
  - A `refuse` whose restart failed leaves the dobby-stop marker standing with no land request. The changed text already covers it: the re-check refuses again or lands under the final ready record, and with no ready record the lifted-hold branch restarts the dobby.
  - An old binary's accept order removes the land request first. A crash after that removal leaves the marker and the Parent-Align hold. The retired-row move and resume then reach the third detection, which settles it.
  - Under the accept's new order, a crash between removals still leaves the land request standing.
  - A running re-step is caught by the board hold as before.
- **F2:** Every `Halt` from `ResolvePullRequest` comes from `checkOrigin`'s missing fact (`internal/landingshed/pullrequestmode.go:87-104`), so `OriginMissing` covers the same inputs. That includes the always-halt case of an unparsable github.com origin under an `on` parent. A `merge` parent with no commits beyond its parent still passes, because the empty-branch answer comes before the origin is judged. The new `merge` refusal row and the `merge` passing row therefore don't conflict.
- **F3:** The `PullRequestSide` seam exists, defined in card 13 (`landDeps` gains `PullRequestSide func(slug string) (beamPullRequestSide, error)`). Refusing on a failed GitHub read matches what Finalize itself needs before it can land.
- **F4:** `success`, `neutral` and `skipped` count as satisfied. Commit-status `error` and `pending` (read as empty) are still treated as owed, which is correct.
- **F5:** `awaiting` is a real shed state (`StateAwaiting` in `internal/shedengine/activity.go:40`). A finished pawl run names Beam-Down, so the not-at-Beam-Await refusal covers it.
- **F6:** The doc rewrites conflict with no sibling card text.

Calls:
- I grepped `PullRequestSide` in the plan and `StateAwaiting` in shedengine. My context had neither card 13/14's seam definition nor whether `awaiting` is a real shed state.
