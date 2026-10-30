# PR-Review row: land a task through a GitHub pull request

The `PR-Review` row, a logging stub until now, lands a task or a beam through a GitHub pull request that the operator discusses with an interactive agent before the run goes on.
Whether a landing uses a pull request is decided by the receiving branch, with a per-entry override.
Hubs with a non-GitHub origin keep landing as they did.

## What changes

- `beam.yaml` gains `pull_request` (`on`, `off`, `required`; template default `on`), read from the receiving branch's worktree: the prime for `main`, the beam's worktree for a beam.
  A board task or project gains a `pull_request` field (empty, `on`, `off`).
  `required` cannot be turned off by the field; `off` under `required` is overridden with a Warn.
  The one resolver, `ResolvePullRequest` in `internal/landingshed`, serves both the row and Finalize (`PATTERN-pull-request-mode`).
- The row pushes the task branch, opens or finds the pull request, sets its title and body from the change description, and records it through the new `internal/pullrequest`.
  It then starts one interactive session on landing.yaml's `pr_reviewer` (default `opus[high]`) with a new stencil, `landing-template-pr-review.md`.
  The session is denied `git push` and `gh` by a new `shellguard` row.
- The agent may commit to the task branch.
  The row gates each clean new head with the new `pr-review` gate key (tier 0, diff scope) and pushes it on a pass; a failure is mailed to the agent once per head.
- The row waits on the pull request without the parent lock.
  A review ends when another person approves it (on the gated head, no standing change request), when it is merged or closed on GitHub, or when the operator asks the agent to continue.
- `lym loom pr-continue`, run by the agent, sets a flag in the pull-request record.
  The row then closes the pull request without merging it, and the run goes on through the ordinary Finalize with its gate and trailer.
- Finalize, under the parent lock, re-runs the resolver.
  With an open pull request it pushes, merges on GitHub with the pushed head as the expected head (squash per `squash`), fast-forwards the parent pair, and lands through the already-landed path.
  A pull request merged elsewhere takes that path directly.
  A yes with no record, a closed pull request, and a refused merge are each a halt with a way forward.
- `pairteardown` closes the run's open pull request, with a comment naming the cause, on every teardown of a present task worktree.
- The quiet check (`internal/battencli`) reads a child status running at `PR-Review` as a live wait, like `Beam-Await`.
  The row also sets the driver strand's wait mark to `PR #<n>`.
- `loom.pr_review`, `darn.pr_review` and the `pr_review_setting` row key are retired.
  The pawl recipe gains a `PR-Review` row between `Beam-Describe` and `Finalize`.
- `githubclient.ParseOwnerRepo` reads the `ssh://git@github.com/...` and `https://<user>@github.com/...` shapes.
  Shuttle gains an unawaited start that carries a prompt, and a resume by session id.

Closes nothing on the board's issue list; the board entry lists no issues.

## Design decisions

- There is no `pr_account` setting.
  The pull request is opened as the gh user, through `internal/githubclient`'s existing token chain; no production code runs `gh`.
  GitHub refuses an approval from the pull request's author, so the operator's ending is the agent's `pr-continue`, which keeps Finalize's gate and trailer.
- A non-GitHub origin, or a parent without an upstream, makes a parent-derived `on` a no.
  A parent `required` or an entry field `on` halts instead, so an explicit demand never lands without a pull request.
- A pull request merged on GitHub lands only if its head equals the head the row gated; any other head halts.
  The shell-guard denies are a guardrail, not a barrier.
- A pause covering the run stops the session and holds the row in process until it lifts; it does not end the call with a transient mark.
- The wait has no timeout and holds one dobby `max_runs` slot per waiting run.
- Default `on` changes landing for every GitHub-hosted hub on its first run after this ships, this hub included; a hub that should keep landing unattended sets `main`'s `pull_request` to `off`.
- A pull request merged on GitHub by someone else carries GitHub's commit message, so the parent's commit lacks lym's trailer block; that landing writes no merge record on the records side.

## To check by hand

- The row's wake loop in `internal/landingshed/prreview.go` gates a moved head; confirm each wake reads the pull request's status before it gates, so a merged or closed pull request is not gated.
- The wait's once-a-minute GitHub poll bound: check that a continue flag set by `pr-continue` is noticed on a wake and not only on a poll, so the bound does not delay the operator's ending by up to a minute.
- `internal/landingshed`'s import allowlist in `seam_enforcement_test.go`: confirm it lists the packages the row now imports and no stale entry.
- Branch protection that dismisses stale approvals turns a Finalize that brought the branch level into a halt and a second approval.
- The real GitHub path was not run: opening, closing and merging a pull request, and a live interactive session with `pr-continue`, are covered by seam tests only.
- The gate's one failure, `TestCLIForm_EveryCommand` (`lym board upsert` help over the 2000-byte cap), was fixed in `8ee8c1a2`; re-run `lym gate test ./cmd/lym` to confirm it is clear.
