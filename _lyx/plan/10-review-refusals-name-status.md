# Card 10 — review-refusals-name-status

**Create:**
- `plan:internal/loomcli#reviewStatusWay` -> `func reviewStatusWay(slug string, bySlug bool) string`

**Edit:**
- `internal/loomcli#reviewWayNoOpen`
- `internal/loomcli#reviewStoreErr` -> `func reviewStoreErr(out io.Writer, verb, statusWay string, err error) int`
- `internal/loomcli#reviewWayNotAtCap` -> `func reviewWayNotAtCap(lead, statusWay string) string`
- `internal/loomcli#reviewNotifyVerb` -> `func reviewNotifyVerb(out io.Writer, store parentreview.Store, slug, statusWay string) int`
- `internal/loomcli#reviewDeliveredVerb` -> `func reviewDeliveredVerb(out io.Writer, store parentreview.Store, slug, statusWay, failedReason string) int`
- `internal/loomcli#reviewApproveVerb` -> `func reviewApproveVerb(out io.Writer, store parentreview.Store, slug, statusWay, reviewFile string, readStatus reviewStatusReader) int`
- `internal/loomcli#reviewSupersedeVerb` -> `func reviewSupersedeVerb(out io.Writer, store parentreview.Store, slug, statusWay, reviewFile string, readStatus reviewStatusReader) int`
- `internal/loomcli#reviewRejectVerb` -> `func reviewRejectVerb(out io.Writer, store parentreview.Store, slug, statusWay, reviewFile string) int`
- `internal/loomcli#reviewState`
- `internal/loomcli#reviewState.preRun`
- `internal/loomcli#loomCLI.reviewCmd`
- `internal/loomcli#TestReviewVerbs_Success`
- `internal/loomcli#reviewVerbCalls`
- `internal/loomcli#TestReviewVerbs_NoOpenRequest`
- `internal/loomcli#TestReviewVerbs_ReviewFileMissingOrEmpty`
- `internal/loomcli#TestReviewApprove_SupersedesCapRejectOnBlockedRun`
- `internal/loomcli#TestReviewApprove_SupersedeRefusals`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
D5: the no-open-request refusal and the supersede refusals of `lyx loom review` name a status command chosen by whether a slug was given, not by where the verb runs.
`lyx loom status` takes no slug, so it cannot address another worktree's run from the prime, where the orch runs these verbs, or from a sibling task worktree.

`reviewStatusWay` returns `"lyx batten status <slug>" run from the prime`, with the real slug, when `bySlug` is set, and `"lyx loom status"` otherwise; with no slug only a task worktree addressing itself reaches a verb.
`reviewState` gains `statusWay`, which `reviewState.preRun` sets from `reviewStatusWay(target.WorktreeName, reviewSlugArg(cmd.Name(), args) != "")`.
`loomCLI.reviewCmd`'s four `RunE` closures pass `st.statusWay` to their verbs.

`reviewWayNoOpen` becomes a format whose status command is filled in: `way forward: only Discussion-Write's parent-review gate opens a request; %s shows whether the run has reached it`.
`reviewStoreErr` takes `statusWay` and fills it into the no-open-request message; its other branches are unchanged.
`reviewWayNotAtCap` takes `statusWay` in place of the fixed `"lyx loom status <slug>"`.
Each verb takes `statusWay` and passes it to `reviewStoreErr`, and `reviewApproveVerb` passes it on to `reviewSupersedeVerb`, which passes it to `reviewWayNotAtCap` in each of its three refusals.

Tests:
`TestReviewVerbs_NoOpenRequest` runs each verb twice, with `reviewStatusWay("task", true)`, whose message names `lyx batten status task` and `from the prime`, and with `reviewStatusWay("task", false)`, whose message names `lyx loom status` and no `<run>`.
`TestReviewApprove_SupersedeRefusals` asserts the same pair on its not-blocked and not-at-cap refusals.
The other listed tests and `reviewVerbCalls` only pass a status way at their call sites.

`refusal-spec.md`'s `review: no open request`, `review: approve over a reject, run not halted` and `review: latest round is not the cap's reject` rows name the status command as: with a slug, `lyx batten status <slug>` run from the prime; with none, in the task worktree, `lyx loom status`.

**ImpactSummary:** the review verbs, their shared error mapper and their tests gain one status-way argument; only the way-forward text of three refusals changes.
