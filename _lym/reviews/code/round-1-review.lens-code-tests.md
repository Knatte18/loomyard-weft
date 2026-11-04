```yaml
lens: code-tests
outcome: reported
```

**Per-behavior list (code-tests lens)**

| Behavior the diff adds or changes | Pinning test | Result |
|---|---|---|
| `LoadConfig` accepts `merge`, and the refusal names all four values | `TestLoadConfig` rows `a worktree layer's pull_request of merge resolves as set` and `an unknown pull_request is refused…` | pinned |
| `ResolvePullRequest`: `merge` is open for every entry field | `TestResolvePullRequest/ModeAndFieldCombinations`, rows `{merge, "", on, off}` | pinned |
| `MergeAtOnce` is set only for `merge` | same test, `wantAtOnce` assertion | pinned |
| Warning for an off field under `merge` | same test, `wantWarn` | pinned |
| A missing upstream under `merge` halts and names the demand | `NoUpstreamUnderMergeHalts` | pinned |
| `PRReview.decide` returns Done at once for `merge`, with no record, push or GitHub call | `TestPRReview_Call` row `AMergeParentLeavesThePullRequestToFinalize…` | pinned |
| Finalize with no record: opens the PR, squash-merges it with `squash` off, and runs fast-forward, merge, mark and push in order | `AMergeParentWithNoRecordOpensThePullRequestAndSquashMerges…` | pinned. The `wantGitHubMerge` call string carries the squash flag. |
| `openAtOnce` finds an open PR and edits it instead of creating one | `AMergeParentFindsTheOpenPullRequestOnTheTaskBranch…` | pinned |
| `openAtOnce`: `Create` fails, so Finalize halts and merges nothing | `AFailedOpenUnderAMergeParentHalts…` | pinned |
| Method refusal under `atOnce` names the repository settings, not `lym config landing` | `AMethodRefusalUnderAMergeParent…` | pinned. It also covers `atOnce` on an existing open record and `squash` forced on. |
| `pushParent` with a plain protected-branch refusal gives the merge way forward, with no repair note | `TestFinalize_MergeMarkPushOrderAndVerdict` row `a protected branch refusal names…` | pinned |
| `pushParent` with a `PushFailedError`-wrapped repository-rules refusal replaces the class's way forward | row `a classified repository-rules refusal…` | pinned |
| Edited repair-note assertion (`wantNote`) | same test | the assertion is still checked |
| `merge` in the `beamModeUnreadableReason` text | none | static prose, exempt |
| Template, `doc.go` and comment text | none | static prose, exempt |
| **Recorded closed PR with `ReviewEnded` under `merge`: reopen, then `atOnce` forces squash** (`finalize_pr.go:212`) | none | **gap** |
| `openAtOnce` writes the record with `GatedHead`, `ReviewEnded`, `Base` and `Head` | none beyond `wantRecordKept`, which checks only that the file exists | **gap** |
| `openAtOnce`: `FindOpen` fails, `Edit` fails, or `WriteRecord` fails | none | gap, only the `Create` failure is covered |
| Transient precedence: a transient push failure is returned before the protected-branch check | none new for protected text | gap |
| `protectedBranchRefusal`: negative case, a non-protected failure that keeps the old reason | the existing non-protected rows (`remote rejected`) in the same test | pinned |
| Marker matching is case-insensitive | none | trivial, not a finding |

**Findings**

1. **MEDIUM, design.** `internal/landingshed/finalize_pr.go:212`. The diff added `atOnce: decision.MergeAtOnce` to the reopen branch. The only reopen rows (`finalize_pr_test.go:164`, `:170`) use `parentMode: "on"`. Dropping `atOnce` there would make a reopened PR merge with `landing.yaml`'s `squash` under `merge`, and no row would fail.
   Fix: add one row with `parentMode: "merge"`, a recorded `reviewEnded` closed PR and `squashOff: true`, expecting `Reopen 7` and then `wantGitHubMerge`.

2. **MEDIUM, design.** `internal/landingshed/finalize_pr.go:330-336`. The record `openAtOnce` writes is not asserted. The design decision says the re-entry relies on `ReviewEnded: true` and the pushed head as `GatedHead`. `wantRecordKept` only checks `found` (`finalize_pr_test.go:361`).
   Fix: have the two open-path rows read the record and compare its `Number`, `Base`, `Head`, `GatedHead` and `ReviewEnded`. The change is an assertion on the existing rows, not a new test.

3. **LOW, design.** `internal/landingshed/finalize_pr.go:318-328`. Two `openAtOnce` failure paths have no row. `FindOpen` and `Edit` failures would be judged through `githubFailure`, but no case reaches them, and the record-write error path is also uncovered.
   Fix: add rows to the same table, or accept `AFailedOpen…` as sharing the same `failed` closure.

4. **LOW, design.** `internal/landingshed/finalize.go:630-633`. No row sends a transient push failure whose text contains `protected branch`, so the order of `transientFailure` before `protectedBranchRefusal` is unpinned. The risk is small because both marker texts are GitHub's permanent rejections.

**Test economy**
- No new test duplicates an existing case.
- The new rows extend the existing tables, as the rubric asks.
- No assertion was added on static prose.

**Tool calls:** one `grep` over `finalize_pr_test.go` for `wantRecordKept`, `GatedHead`, `ReviewEnded`, `Reopen` and the `merge` rows. My context held only the diff hunks of that file, not its remaining rows.
