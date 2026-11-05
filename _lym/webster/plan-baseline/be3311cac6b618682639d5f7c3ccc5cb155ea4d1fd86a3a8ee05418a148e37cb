# Card 3 — pr-merge-state

**Edit:**
- `internal/pullrequest#Status`
- `internal/pullrequest#Client.Status`
- `internal/pullrequest#API`
- `internal/pullrequest/doc.go`
- `internal/pullrequest_test#TestClient_Calls`
- `internal/landingshed#fakePullRequestAPI`
- `internal/pairteardown#closeRecordingAPI`

**Create:**
- `plan:internal/pullrequest#Check` -> `type Check struct`
- `plan:internal/pullrequest#Client.RequiredChecks` -> `func (c *Client) RequiredChecks(ctx context.Context, base, head string) ([]Check, error)`
- `plan:internal/pullrequest#MergeState` -> `type MergeState struct`
- `plan:internal/pullrequest#MergeRetryWindow` -> `const MergeRetryWindow = time.Minute`
- `plan:internal/pullrequest#ReadMergeState` -> `func ReadMergeState(ctx context.Context, api API, number int, beamHead, base string, now func() time.Time, sleep func(ctx context.Context, d time.Duration) error) (MergeState, error)`
- `plan:internal/landingshed#fakePullRequestAPI.RequiredChecks` -> `func (f *fakePullRequestAPI) RequiredChecks(_ context.Context, base, head string) ([]pullrequest.Check, error)`
- `plan:internal/pairteardown#closeRecordingAPI.RequiredChecks` -> `func (a *closeRecordingAPI) RequiredChecks(context.Context, string, string) ([]pullrequest.Check, error)`
- `internal/testkit/prkit/doc.go`
- `plan:internal/testkit/prkit#API` -> `type API struct`
- `plan:internal/testkit/prkit#API.FindOpen` -> `func (a *API) FindOpen(ctx context.Context, head, base string) (pullrequest.Ref, bool, error)`
- `plan:internal/testkit/prkit#API.Create` -> `func (a *API) Create(ctx context.Context, head, base, title, body string) (pullrequest.Ref, error)`
- `plan:internal/testkit/prkit#API.Edit` -> `func (a *API) Edit(ctx context.Context, number int, title, body string) error`
- `plan:internal/testkit/prkit#API.Status` -> `func (a *API) Status(ctx context.Context, number int) (pullrequest.Status, error)`
- `plan:internal/testkit/prkit#API.Close` -> `func (a *API) Close(ctx context.Context, number int, comment string) error`
- `plan:internal/testkit/prkit#API.Reopen` -> `func (a *API) Reopen(ctx context.Context, number int) error`
- `plan:internal/testkit/prkit#API.SetDraft` -> `func (a *API) SetDraft(ctx context.Context, number int, draft bool) error`
- `plan:internal/testkit/prkit#API.Comment` -> `func (a *API) Comment(ctx context.Context, number int, body string) error`
- `plan:internal/testkit/prkit#API.Merge` -> `func (a *API) Merge(ctx context.Context, number int, expectedHead string, squash bool, title, message string) error`
- `plan:internal/testkit/prkit#API.RequiresPullRequest` -> `func (a *API) RequiresPullRequest(ctx context.Context, branch string) (bool, error)`
- `plan:internal/testkit/prkit#API.RequiredChecks` -> `func (a *API) RequiredChecks(ctx context.Context, base, head string) ([]pullrequest.Check, error)`
- `plan:internal/pullrequest_test#TestReadMergeState` -> `func TestReadMergeState(t *testing.T)`

**Uses:**
- `internal/pairteardown/doc.go`

**Intent:** Read a pull request's merge state the way D6's `lym beam pr status` and D3's refusal 6 judge it, in one function both call, and give the beam verbs' tests one shared fake of the GitHub calls.

- `Status` gains `MergeableState string`, GitHub's REST `mergeable_state` as go-github reads it (`clean`, `dirty`, `blocked`, `behind`, `unstable`, `has_hooks`, `unknown`, `draft`; Constraints), filled by `Client.Status` from the pull request it already gets.
- `API` gains `RequiredChecks(ctx, base, head)`: the status checks the repository's rules for base require, each with its conclusion at head. `Check` holds `Name` and `Conclusion` (empty while the check has not finished, `missing` when neither a check run nor a commit status of it exists at head). `Client.RequiredChecks` reads every page of `Repositories.GetRulesForBranch` for base, collects the required status-check contexts, then reads every page of `Checks.ListCheckRunsForRef` at head and matches each context by name. It also reads every page of `Repositories.GetCombinedStatus` at head and takes a context's conclusion from its commit status when no check run carries it, the status's `pending` read as empty since it has not finished, so a required context a CI reports as a commit status is read; a context is `missing` only when neither a check run nor a commit status carries it. The landingshed and pairteardown fakes implement it returning nothing.
- `MergeState` holds `State`, `Draft`, `Head`, `MergeableState`, `Approved`, `Checks []Check`, `MergeableNow bool` and `Causes []string`.
- `ReadMergeState` reads `Status` and `RequiredChecks` and judges: `MergeableNow` is true only for an open, non-draft pull request whose head is beamHead and whose `MergeableState` is `clean`, `has_hooks` or `unstable`. Otherwise each cause is named: `closed` or `merged` for a pull request no longer open, `draft`, `head <sha> is not the beam's head <sha>`, `dirty` as a conflict, `blocked` as a review or required check still owed (naming each required check whose conclusion is not `success`, `neutral` or `skipped`, the conclusions GitHub counts as satisfied for a required check; an empty or `missing` conclusion, `failure`, `cancelled`, `timed_out`, `action_required` and `stale` are owed), `behind` as a base ahead of the head. `unknown` is read again through sleep, doubling from one second, until `MergeRetryWindow` of the now clock has passed since the first read, then reported with the cause `GitHub has not yet computed whether it can merge`. A failed GitHub call is returned.
- `internal/pullrequest/doc.go`'s first sentence stops naming PR-Review, Finalize and pair teardown as the callers and names no caller list, since the `lym beam pr` verbs and the retired-row move call the package too.
- `MergeRetryWindow` is the length of Finalize's merge retry window (D6); card 11 points `landingshed`'s `prMergeRetryWindow` at it.
- `internal/testkit/prkit` is the shared test kit (`PATTERN-test-isolation`): `prkit.API` implements `pullrequest.API` with a scripted answer and a recorded call list per method, guarded by a mutex, and its `doc.go` says so. Cards 13 to 15 drive the beam verbs through it.

Tests (tier 0): `TestClient_Calls` gains rows for the `mergeable_state` read and for `RequiredChecks` over a scripted rules page and check-runs page (a passing, a failing and a missing check), and a row whose required context is reported only as a commit status, read with that status's conclusion and not as missing. The new `TestReadMergeState` drives a `prkit.API` with a fake clock: an open clean pull request at the beam's head is mergeable now; `has_hooks` and `unstable` are too; each of draft, head mismatch, `dirty`, `blocked` with a failing required check, `behind`, closed and merged names its cause and is not; a `blocked` pull request whose one required check concluded `skipped` and another is owed names only the owed one; `unknown` turning `clean` within the window is mergeable now; `unknown` past the window names the not-yet-computed cause and slept until the window ended; a failing `Status` call is returned.

**ImpactSummary:** pullrequest's API grows one method, so its two test fakes in landingshed and pairteardown gain it; `Status` grows a field no caller sets; the merge-state judgment and the shared fake kit are new.

**Commit:** 3: Read a pull request's merge state and required checks
