```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests)**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `IsMergeTaskWriter` / `IsMergeTaskSlug` (role, slug suffix, numbered, suffix inside slug) | `TestIsMergeTaskWriter` in `mergetask_test.go` | Pinned. |
| Guard lets a merge task's darn writer run `merge-in`, `merge-stage`, `merge --continue`, `merge --abort` | `roleguard_test.go` subtest "a merge task's writer runs the merge verbs and no other mutating form" | Pinned. |
| Guard still refuses `merge <branch>`, `push`, `commit` for that writer, and every verb for other tasks and other roles | Same subtest | Pinned. |
| `mergeParent` files `pull_request: merge-in` | `TestMergeParent` in `mergeparent_test.go` | Pinned. |
| `mergeParent` uses `darnengine.MergeTaskSuffix` and `IsMergeTaskSlug` for slug cut and open-task lookup | `TestMergeParent` (the long-slug row uses the suffix) | Pinned. |
| `normalizeTask` accepts `merge-in`, and the refusal text names it | `TestNewTask` in `task_test.go`, the pull_request subtest | Pinned. |
| `ResolvePullRequest` answers no with `TrueMerge` under every parent mode and rule | `MergeInFieldWantsNoPullRequestUnderAnyParentOrRule` in `pullrequestmode_test.go` | Pinned. |
| A beam ignores the `merge-in` field | The last assertion of the same subtest | Pinned. |
| `resolveWithParentRule` skips the GitHub rule read for `TrueMerge` | Row "AMergeInEntry...NoRuleRead..." in `finalize_pr_test.go` | Not pinned. The row name claims it, but the visible row sets no `wantCalls` or rule-read assertion. Resolution gives the same answer with or without the read, so removing the `decision.TrueMerge` short-circuit would pass every test. |
| Finalize's parent-side merge drops the squash for `TrueMerge` | Same row, `wantTrueMerges` | Pinned. `deps.Config.Squash = !tt.squashOff` (line 451) defaults to true, so removing `&& !landing.trueMerge` fails the row. |
| `pushParent` omits the `pull_request=merge` advice on a protected-branch refusal for a true merge | Row "AProtectedParentPushRefusalOfAMergeInEntry..." | Pinned. |
| `pullRequestLanding` sets `landing.trueMerge` from the decision | Same two rows | Pinned. |
| PR-Review passes at once on `merge-in`, opens nothing, closes any recorded pull request | None | Not pinned. `prreview_test.go` has no `merge-in` row. Only the resolver is tested, not PR-Review's `!decision.Open` path with that field. |
| `recordedBeamLanding`, the already-landed path and the reopen path for a `merge-in` entry | None | Not pinned. This is partly moot, since a beam ignores the field and `trueMerge` is set only on the non-beam path. |
| The landing gives the beam the parent's commit as an ancestor (issue #66) | None | Not pinned. The Finalize row uses a fake merger and asserts only `opts.Squash == false`. No test runs a real parent-side merge with `Squash` false and checks ancestry. |
| `mergeParentBody` wording | `TestMergeParent` checks only two substrings | Prose is exempt. The text at `mergeparent.go:126` has no space before the backtick, and nothing would catch it. That is a defect for the handler, not a coverage gap. |

**Findings**

1. **MEDIUM, design.** `internal/landingshed/prreview_test.go` (no row exists; the code is `prreview.go:101-102`). Nothing tests PR-Review's behaviour for a `merge-in` entry. The answer key's "no PR-Review row and no GitHub pull request" is carried only by the resolver, and PR-Review's `!decision.Open` branch (`closeRecordedPullRequest`, `prreview.go:101-102`) is never run with `merge-in`. Add a PR-Review row that sets `EntryField` to `merge-in` and asserts Done, no pull request opened, and no GitHub rule read. The existing field helper is at `prreview_test.go:660`.
2. **LOW, design.** `internal/landingshed/finalize_pr_test.go:229` (code at `pullrequestmode.go:159`). The row name promises "NoRuleRead", but no assertion backs it, and the `decision.TrueMerge` short-circuit in `resolveWithParentRule` can be deleted with every test still green. Assert that the rule-read seam is not called for the `merge-in` row, or drop the claim from the name.
3. **LOW, design.** `finalize_pr_test.go:229` (issue #66 behaviour). The landing is tested only against a fake merger, so nothing shows the parent-side merge with `Squash: false` leaves the merged commit as an ancestor of the beam. If an existing `fabricengine` or `landingshed` integration test already covers an unsquashed landing, the handler can reject this. I found none in the diff.

Tool calls (1):
1. `grep` of `finalize_pr_test.go` and `prreview*_test.go` for `Squash`, rule-read and `EntryField`, because my context lacked how `Config.Squash` is set in the test and whether PR-Review has a `merge-in` row.
