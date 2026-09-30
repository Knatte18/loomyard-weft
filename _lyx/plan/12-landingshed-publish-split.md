# Card 12 — landingshed-publish-split

**Edit:**
- `internal/landingshed/publish.go`
- `internal/landingshed/doc.go`
- `internal/landingshed/publish_test.go`
- `internal/landingshed/publish_integration_test.go`

**Delete:**
- `internal/landingshed/publish_awaiting_test.go`

**Uses:**
- `internal/landingshed/prgate.go`
- `internal/summaryparser#Parse`

**Intent:**
Take the review wait out of Publish now that the gate row owns it (D2, D7).
In `Publish.Call`, delete step 3a and `checkApproval`: Publish no longer reads the approval record.
After the push and the pull-request query:
- no pull request → create it from the change description, as today, and return `Done`;
- an open pull request → parse the change description, and when its title or body differs from the pull request's, edit them through `client.PullRequests.Edit` under `publishGitHubTimeout`, then return `Done`;
  an edit failure takes the same transient-error-or-`Stuck` split as the create call;
- merged → `Done`; closed unmerged → `Stuck`, both unchanged.
Delete `awaitingOrCancelled`, which has no caller left; the gate carries its own.
Rewrite `Call`'s and the file's doc comments to say the Done after an open PR is safe because the next row is the gate.
Rewrite `doc.go`'s Publish paragraphs: Publish syncs, verifies, pushes and opens or refreshes the pull request (title and body included), and the `PR-Gate` producer in the same package then owns approval, rejection and the wait, per D2's table, stated by pointing at `PRGate.Call` rather than restating it.
Tests: in `publish_test.go`, the created-PR and open-PR cases now expect `Done`;
add an open PR whose title or body differs from the description (one Edit request carrying the new values) and one whose both match (no Edit request);
remove the approval cases (`newApprovalFixture` and its tests), whose behaviour card 7's gate tests cover.
Delete `publish_awaiting_test.go`, whose Awaiting cases no longer exist and whose closed-PR case `publish_test.go` already covers.
`publish_integration_test.go`'s `TestPublish_MergesInCleanlyBeforeCreatingPullRequest` expects `Done` after the create.

**ImpactSummary:** Publish returns Done where it returned Awaiting and stops reading the approval record; the recipe already routes its Done to the gate.
