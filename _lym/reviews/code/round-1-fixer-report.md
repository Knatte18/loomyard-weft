---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## What changed

- F1 (`internal/gitrepo/pull.go`, `doc.go`, `fetch_integration_test.go`): `Repo.Fetch` runs the plain `git fetch`, then under the same remote-refs hold `git fetch --prune --no-tags origin <refspecs>`.
  The refspecs are the remote's configured ones whose destination starts with `refs/remotes/`, and the second call is skipped when none qualifies or `origin` is not configured.
  The error text names the command that failed, with each refspec single-quoted so a paste does not glob.
  The docs say Fetch never prunes a local tag.
  `TestFetch_RecordsRefspecs_PrunesTrackingRefsButKeepsLocalTags` pins the records-store case.
  Commits: `gitrepo: fix F1 — Fetch prunes only refs/remotes/ destinations, never a local tag` and a follow-up for the quoting and the remote-name doc.
- F2 (`_lym/landing/summary.md`): Merge danger is one-way with the wider blast radius.
  The what-changes and tests sections now match F1, F3, F4 and F5.
  Not committed: the file lies under `_lym`, which this round never touches with git.
- F3 (`internal/battencli/arm.go`): the `hubRunVerdict` fetch closure also fetches the prime's code store, and its doc comment says so.
- F4 (`internal/battencli/arm_runverdict_test.go`): `readErr` became a per-read `readErrs` list, with a row for a read that fails after the fetch.
- F5 (`internal/runfsm/act_test.go`): a row pins the pair-absent refusal with no tip and no machine.

## Self-check

- fix-consistency, MEDIUM: summary first bullet still said bare `--prune` — fixed in `summary.md`.
- fix-consistency, MEDIUM: summary omitted the prime code-store fetch — fixed in `summary.md`.
- fix-consistency, LOW: summary Tests section stale — fixed in `summary.md`.
- fix-consistency, LOW: summary lacks why the prune is limited to `refs/remotes/` — fixed, the first bullet now says so.
- fix-affected-set, LOW: prune reads `origin` only — fixed differently: the constant's doc comment states that a checkout with no `origin` is fetched but never pruned; every store in the repo uses `origin`, and reading the branch's upstream remote would reach beyond the finding.
- fix-proposal, LOW: same `origin` point — same disposition.
- fix-code-correctness, LOW: same `origin` point — same disposition.
- fix-proposal, LOW: the reproduction command's refspecs unquoted — fixed, each refspec is single-quoted.
- fix-proposal, LOW: the F3 fetch repeats the code-side fetch when a pair stands — fixed as proposed, no change: a redundant round trip on a rare path, and a guard would move `catchUpPair` knowledge into the closure.
- fix-code-correctness, LOW: the new fetch test's clone also loses `refs/remotes/origin/main` — no change: harmless and not asserted, and the rubric excludes it.
- All four forks returned reports.
- Gate finding: `TestCheckedCallInvariant_RawSitesMarkedAndPinned` found four raw sites in `internal/gitrepo`, pinned at three — fixed, `Fetch` now has two marked raw calls, so `cmd/lym/checkedcall_test.go` pins four.
- Gate finding: the comment on `TestFetch_RecordsRefspecs_PrunesTrackingRefsButKeepsLocalTags` wrapped mid-sentence — fixed, it is now one sentence per line.

## Disputed

None.

## Deferred

None.

## Rulings

None.
