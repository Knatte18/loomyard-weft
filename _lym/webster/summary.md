# PR-Review row: landing through a GitHub pull request

All six batches reached `done`.
No batch reported renames or `Pattern-Departure` trailers.

## Batches

- **01 (cards 1–5), head 71215e0f:** `ParseOwnerRepo` reads the `ssh://` and user-info `https://` origin shapes.
  `beam.yaml` gains the `pull_request` mode, the board entry gains its override, and `landing.yaml` gains `pr_reviewer` and the `pr-review` gate key.
  Shuttle starts an unawaited session resumable by session id.
  The fork made `allOutputFilesExist` false for an empty output list instead of changing `stopOutcome`, as card 5 had it, so a provider that died at startup does not read as started.
  Deviations listed: `internal/githubclient/parseownerrepo.go`, `internal/shuttleengine/stop.go`.
- **06 (cards 6–9), head fad3f508:** the `publish` shell-guard row on `Spec.DenyPublish`, the `internal/pullrequest` record and client, landingshed's `ResolvePullRequest` with the `PATTERN-pull-request-mode` entry, and `lym loom pr-continue`.
  The fork followed card 7's explicit mapping, so a moved head is `RefusalOther` rather than pending.
  Deviations listed: `internal/landingshed/pullrequestmode.go`, `internal/loomcli/prcontinue.go`, `internal/pullrequest/client.go`, `internal/shellguard/rows.go`.
- **10 (cards 10–11), head 91820d42:** the PR-Review stencil, and the row that decides, pushes, opens or finds, and records the pull request.
  A recorded pull request that GitHub reports closed or merged is left as it is and the row returns Done.
  Deviations listed: `internal/landingshed/prreview.go`, `internal/landingshed/prreview_test.go`, `internal/loomcli/landingdeps.go`, `internal/loomrecipe/pawlrecipe_test.go`, `internal/shedrecipe/entries_prreview.go`.
- **12 (cards 12–13), head 57c0e718:** `loom.pr_review` and `darn.pr_review` are retired keys, and PR-Review waits on the pull request, gates agent commits and ends on its state.
  A head whose gate failure was already mailed is not gated again at each wake.
  Deviations listed: `internal/landingshed/prreview.go`, `internal/landingshed/prreview_test.go`, `internal/landingshed/prreview_wait_test.go`, `internal/landingshed/verifygate.go`, `internal/loomcli/landingdeps.go`.
- **14 (cards 14–16), head bb4dfd77:** PR-Review keeps one interactive session per run, Finalize lands through the pull request under the parent lock, and pair teardown closes the run's open pull request.
  The fork's card 16 gate and batch gate exited 1 on `TestCLIForm_EveryCommand`: `lym board upsert` has a Long of 2098 bytes against a cap of 2000.
  Card 3's help text in batch 01 caused it, and the fork left it for the `webster` gate's fixer.
  Deviations listed: `cmd/lym/ghguard_test.go`, `internal/landingshed/finalize_pr.go`, `internal/landingshed/prreview.go`, `internal/landingshed/prreview_session_test.go`, `internal/landingshed/prreview_test.go`, `internal/landingshed/prreview_wait_test.go`, `internal/landingshed/prsession.go`, `internal/loomcli/landingdeps.go`, `internal/loomcli/landingdeps_test.go`, `internal/pairteardown/pullrequest.go`, `internal/pairteardown/teardown_test.go`, `internal/testkit/envkit/envkit.go`.
- **17 (card 17), head f2af6eb1:** the quiet check reads a running status at `PR-Review`, like `Beam-Await`, as a live wait.
  No deviations.

## Verify gate findings

- `github.com/Knatte18/loomyard/cmd/lym.TestCLIForm_EveryCommand` failed (attempt 1 of 2), in a package card 14's commits touched.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `8ee8c1a21bd5f838dab67c08c0a3b6558477ee01` fix: shorten lym board upsert help under the 2000-byte Long cap
