# Card 14 — beam-pr-status-close

**Create:**
- `plan:internal/beamcli#prStatus` -> `func prStatus(ctx context.Context, deps landDeps, slug string) (map[string]any, error)`
- `plan:internal/beamcli#prClose` -> `func prClose(ctx context.Context, deps landDeps, slug string) (map[string]any, error)`
- `plan:internal/beamcli#TestPRStatus` -> `func TestPRStatus(t *testing.T)`
- `plan:internal/beamcli#TestPRClose` -> `func TestPRClose(t *testing.T)`

**Edit:**
- `plan:internal/beamcli#prCmd`
- `internal/beamcli#landDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#newLandWorld`
- `internal/beamcli/doc.go`

**Uses:**
- `plan:internal/pullrequest#ReadMergeState`
- `plan:internal/pullrequest#MergeState`
- `internal/landingshed#ResolvePullRequest`
- `plan:internal/landingshed#BeamWithoutPullRequest`
- `plan:internal/beamcli#judgePullRequestVerb`
- `internal/beamcli#judgeLandEntry`
- `plan:internal/beamcli#beamPullRequestSide`
- `plan:internal/testkit/prkit#API`

**Intent:** Add `lym beam pr status <slug>` and `lym beam pr close <slug>` (D6).

- `landDeps` gains `BeamHead func(slug string) (string, error)` (the beam pair's HEAD), `PullRequestMode func(slug string) (mode string, rulesRequired bool, err error)` (the parent's beam.yaml `pull_request`, read as loomcli's `parentPullRequestMode` reads it, and whether GitHub's rules for the parent branch require a pull request, through the API's `RequiresPullRequest`), `Now` (already there) and `Sleep func(ctx context.Context, d time.Duration) error`; `hubLandDeps` fills them.
- `prStatus` is read-only and answers at any row up to Beam-Down: it runs `judgeLandEntry` alone, reads the landing record, and with none recorded answers `recorded: false`, naming `lym beam pr open <slug>` while the run waits at Beam-Await and, at any later row, that Finalize decides the landing's pull request by the parent's rules. Otherwise it calls `pullrequest.ReadMergeState` with the beam's head and the parent branch, the same function card 15's refusal calls, and prints `state`, `draft`, `head`, `mergeable_state`, `approved`, each required check with its conclusion, `mergeable_now` and, when false, the named `causes`; `unknown` is read for up to `pullrequest.MergeRetryWindow` first.
- `prClose` runs `judgePullRequestVerb` and refuses with none recorded, naming `lym beam pr open <slug>`. It reads the recorded pull request's state: an open one is closed with one comment, others skip the GitHub call; then the pull request is dropped from the landing record and the directory committed through `WriteLanding`. The comment and report come from `landingshed.ResolvePullRequest(slug, mode, "", landingshed.BeamWithoutPullRequest, rulesRequired, origin, true)` and the mode: a no (`on` or `off`) comments that the operator chose to land the beam without a pull request; a yes comments only the close. A `merge` parent reports that Finalize will still land through a pull request it finds or opens; a `required` parent, or one GitHub's rules require a pull request on, reports that `lym beam ready` refuses until one is recorded again with `lym beam pr open <slug>`. A failed mode read still closes with the close-only comment and drops, and reports the read's error instead.
- `prCmd` adds `status` and `close` with help texts stating the above; `close` says the orch closes a pull request only to land without one, never to add work, since work lands into the beam while it stays open.
- Bound: `close` acts only on the recorded pull request from the beam's branch, and refuses once the ready record stands; `status` changes nothing.

Tests (tier 0): `TestPRStatus` with a `prkit.API` and a fake clock: none recorded at Beam-Await answers `recorded: false` naming `lym beam pr open`, and at Finalize names Finalize's decision instead; an open clean pull request at the beam's head is `mergeable_now` true; one behind names `behind`; a run at Finalize is still answered. `TestPRClose`, its passing rows first clearing the fixture's default ready flag: an open recorded pull request is closed with the operator's-choice comment under `off`, dropped and committed; a closed or merged one makes no GitHub close call and is dropped; a `merge` parent reports Finalize lands through a pull request; a `required` parent and a rules-required `on` parent each report `lym beam ready` refuses until `lym beam pr open`; a failing mode read still closes and drops and reports the error; no record and a standing ready record are refused naming their ways forward.

**ImpactSummary:** two more `beam pr` verbs and three landDeps seams in beamcli; nothing outside beamcli changes.

**Commit:** 14: Add lym beam pr status and close
