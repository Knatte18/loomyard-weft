---
format: 5
approved: false
language: go
---

# Plan: PR-Review row — land through a GitHub pull request

The `PR-Review` row, today a logging stub, becomes the row that decides whether a landing goes through a pull request, opens or finds it, starts the interactive agent the operator discusses it with, gates and pushes the agent's commits, and waits on the pull request's state; Finalize then merges it on GitHub under the parent lock, or lands locally when the operator ended the review through the agent, and a pair torn down unlanded closes it.
The mode comes from the receiving branch's `beam.yaml` `pull_request`, with a per-entry board override, through one landingshed resolver.

## Card Index

1 — parse-owner-repo-shapes — ParseOwnerRepo reads the ssh:// and user-info https:// origin shapes
2 — beam-pull-request-key — beam.yaml gains the cross-pair pull_request mode on, off or required
3 — board-pull-request-field — the board entry gains a pull_request override, scope under the run lock
4 — pr-review-config-keys — landing.yaml pr_reviewer and the pr-review gate key
5 — shuttle-unawaited-session — shuttle starts an unawaited session with a prompt, resumable by session id
6 — publish-shell-guard — a shell-guard row denying git push and gh on a launch-spec switch
7 — pullrequest-record-and-client — the run's pull-request record and its GitHub calls in internal/pullrequest
8 — pull-request-mode-resolver — landingshed's one resolver of the pull-request mode
9 — pr-continue-verb — lym loom pr-continue, the agent's flag for the operator's ending
10 — pr-review-stencil — the PR-Review session's opening stencil
11 — pr-review-row-opens — PR-Review decides, pushes, opens or finds and records the pull request
12 — retire-pr-review-keys — loom.pr_review and darn.pr_review become retired keys
13 — pr-review-row-waits — PR-Review waits on the pull request, gates agent commits and ends on its state
14 — pr-review-session — PR-Review starts and keeps one interactive session per run
15 — finalize-through-pull-request — Finalize merges the pull request on GitHub under the parent lock
16 — teardown-closes-pull-request — pair teardown closes the run's open pull request
17 — quiet-check-pr-review-wait — the quiet check reads a run at PR-Review as waiting

## Shared Decisions

- **No `pr_account`.** The added-after-Discussion decisions drop D5's named account: every GitHub call goes through `githubclient`'s existing token chain (`GH_TOKEN`, `GITHUB_TOKEN`, its cache, `gh auth token`), so no card adds `pr_account` or `gh auth token --user`, and the D5 acceptance criterion is carried only as its remaining halt: an unresolvable token halts naming `gh auth login --hostname github.com`, then `lym loom resume`.
- **The operator's ending.** The PR-Review agent sets its flag with `lym loom pr-continue` (card 9), which writes `Continue` into the run's pull-request record; the row closes the pull request, records `ClosedForContinue` and is Done (card 13), and Finalize lands locally for that record (card 15), with its gate and trailer. Another person's approval still ends Done and Finalize merges on GitHub.
- **One package for the pull request.** `internal/pullrequest` owns the record, written through `internal/state`, and the GitHub calls over go-github with `githubclient`'s client, like `internal/hubissues`; landingshed's row and Finalize and pairteardown hold its `API` interface, so their tests fake GitHub. The record lives in the run's scratch directory (`shedrun.PullRequestRecord`), never tracked.
- **The session's start.** `Runner.Open` passes no settings, so it cannot install the publish shell guard; the session instead uses shuttle's new unawaited start (card 5), which goes through the engine's prepared settings, so `Spec.DenyPublish` installs the guard (card 6).
- **The blocking push** of D6 is `fabricengine.PushPairAnchored`, the synchronous both-sides push `commit-records` uses (`PATTERN-push-both-sides`).
- **A covering pause holds the row in process.** D9 asks a covering pause to stop the session and end the call with a transient mark, so the shed loop's pause check holds the run and the next call resumes the wait. The mark does not fit `PATTERN-transient-stop` ("Marking a failure transient: only `internal/shedengine`'s mark, translated by `internal/shedtransient` at a producer or step-bootstrap boundary; never on a gate verdict"), whose classes are a closed set of infrastructure failures, none of which a pause is; and a returned `shedengine.Yield` ends the loop and the driver pass with nothing to re-drive the run after the pause lifts (`internal/shedengine/run.go`, `internal/sheddriver/drive.go`). So the row holds in its own call instead, as shuttle's Wait holds a covered run: a covered wake stops a live session and does no gate, push or GitHub call, the wait keeps sleeping, and the first uncovered wake resumes the session and goes on. That is the effect D9 asks for: the session is stopped for the pause and the wait resumes after it, with no new stop class.
- **GitHub failures.** Every caller judges a failed GitHub call with `errors.Is(err, githubclient.ErrTokenUnresolvable)` first, since `http.Client` wraps the token error in a `*url.Error` that `githubclient.IsTransient` reads as transient; then the transient check; then any other failure halts naming the token and repository access.
- **Nil seams.** The row treats a nil pull-request seam as a wiring error; Finalize treats a nil `PullRequest.ParentMode` as no pull request, following the package's nil-is-absent convention, so its existing struct-literal tests land locally; loomcli's `landingDeps` fills both and its drift guard keeps them filled.

## PATTERN changes

- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch, or merging a run's pull request: only in a run's Finalize, under `internal/parentlock`'s lock for that parent.
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize and pair teardown open, merge or close a pull request. (test)
- `PATTERN-pull-request-mode` — adds: Deciding whether a landing goes through a pull request: only landingshed's one resolver, from the receiving worktree's `beam.pull_request`, the landing entry's `pull_request` field and the origin; no profile or run setting carries it.
- `PATTERN-shed-recipe-registry` — changes: Adding producer or recipe row: one `Constructor` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call.
