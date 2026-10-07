# Card 12 — publish-push-rejected

**Create:**
- `internal/loomcli/wiring_publish_integration_test.go`

**Edit:**
- `internal/landingshed/deps.go`
- `internal/landingshed/publish.go`
- `internal/landingshed/doc.go`
- `internal/loomcli/landingdeps.go`
- `contracts/specs/refusal-spec.md`
- `internal/landingshed/publish_test.go`
- `internal/landingshed/publish_integration_test.go`

**Uses:**
- `internal/fabricengine#Fabric.RemoteOnlyCommits`

**Intent:**
Implement decision D8 (#404's Publish half): Publish keeps escalating on `gitrepo.ErrPushRejected` with no retry, and its stuck reason now states what it tried and how to resume.

Add a told closure to `landingshed.Deps`, `RemoteOnlyCommits func() (tip string, commits []string, err error)`, which `internal/loomcli/landingdeps.go` fills by opening the task pair (`fabricengine.Open(l)`) and calling card 10's `Fabric.RemoteOnlyCommits`, the read card 11's reset gate also uses; landingshed itself runs no git (`PATTERN-fabric-git`), and the every-field drift guard in `landingdeps_test.go` proves the field is populated without an edit, while the integration case below proves what it returns.
In `Publish.Call`'s Step 5, on `ErrPushRejected`, call it and build the stuck reason: the merge-in against the parent branch already ran, the remote task branch's tip, and the number of commits it holds that the local branch lacks, then the way forward: merge `origin/<task-branch>` into the task branch in the task worktree, then resume the run with `lyx loom start`.
When the closure itself fails, the reason keeps "push rejected by the remote", says the remote tip could not be read with the cause, and names the same way forward.
The reason ends in the usual `way forward:` clause; no denied command appears in it (`PATTERN-no-denied-recovery`).
Every other push failure keeps its current reason.

Docs: a refusal-spec row "Publish: push rejected" in the loom section, its way forward quoted from the message; `internal/landingshed/doc.go`'s Publish step list.

The regression drives Publish's rejected half through `internal/loomcli`'s real `wire()` seams, as `wiring_rework_integration_test.go` drives PR-Rework, with the production `landingDeps` closure.
The resumed half reaches the GitHub step, which only `landingshed`'s integration tier fakes (`publish_integration_test.go`'s httptest server), so that tier drives both halves of one scenario; this is recorded in the plan's Shared Decisions.

Tests:
`publish_test.go`: `TestPublish_StuckBeforePullRequest`'s row "push rejected has its own reason" is extended so the rejected reason names the faked tip, the faked remote-only count, the merge-then-`lyx loom start` way forward, and that the merge-in ran; a new row where the closure fails still yields a rejected reason with the way forward; the "push failure" row pins a generic push failure's reason unchanged.
`publish_integration_test.go`: on the hubforge pair, push a commit to the remote task branch from a second clone so the task branch's push is rejected; Publish ends stuck with the real remote tip and a count of 1 in its reason, and after merging `origin/<task-branch>` in the task worktree a re-run of Publish pushes and reaches the GitHub step.
`internal/loomcli/wiring_publish_integration_test.go`, a new `integration`-tagged file beside the other `wiring_*_integration_test.go` files: over a hubforge pair whose remote task branch holds a commit the local branch lacks, it calls `wire()` and runs `landingshed.NewPublish(c.env.Landing)`, which halts at the rejected push before any GitHub call, and asserts the stuck reason names the real remote tip, a count of 1 and the merge-then-`lyx loom start` way forward.
It is the one test of the production closure's wiring, which `landingdeps_markdone_test.go` and `landingdeps_test.go` do not reach.

**ImpactSummary:** Publish's push-rejected stuck reason gains the remote tip, the remote-only count and a resume way forward through one new told closure; other Publish paths are unchanged.
