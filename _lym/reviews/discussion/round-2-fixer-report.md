# Fixer report — discussion round 2

Every finding was fixed; none disputed or deferred.
Changes are in `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

## Changes

- F1 (BLOCKING), fixed as proposed: D8 now unsets the branch's upstream config with the tracking-ref drop, in one step under the remote-refs lock; its Rationale cites `UpstreamSHA` and `checkOrigin`.
  D2's `ErrRemoteLeftover` way forward ends with `git -C <records repo> branch --unset-upstream <weft branch>`.
  New D8 criterion: a D1 re-adopt of a local weft branch origin lacks has its pair push land the branch.
- F2 (MEDIUM), fixed as proposed: D1 step 3 lists the fork-point checks the adopt-only option skips and replaces "With both branches adopted, no parent is resolved".
  D2's closed set gains both `refuseAdoptLocalCheckedOut` refusals and the stray-path refusal as typed errors, and the anchor-move stop leaves the transient list.
  D1 states its absent test reads a non-worktree path as absent.
  The new answers get a way forward that holds at every row.
- F3 (MEDIUM), fixed as proposed: D7 gains the bullet that every push of the pair's weft branch outside a removal, `commit-records` included, takes the teardown lock shared.
  The `PATTERN-slug-remote-order` text says "shared for each pair push outside a removal"; after the gate flagged the rule text at 283 characters, over the 280 cap, it was shortened to that wording and drops two articles ("task-branch discard and cleanup sweep").
  New D7 criterion for a `commit-records` push during an exclusive hold.
  "Outside a removal" keeps D4's own push, which runs under the removal's exclusive hold, out of the rule.
- F4 (MEDIUM), fixed as proposed, settled from the evidence: the ref was run A's.
  `originTargets` reads tracking refs before the ls-remote that found origin without the branch, so the ref predates run B's push, and the only earlier weft push in the hub's trace logs is run A's at 08:14:59.
  The ref's reflog starts at 08:21:28, so it cannot show the 08:18:44 state, and the removal's own trace went with run A's worktree.
  The evidence is recorded in the support log's new Round 2 section.
  D8's Rationale stands; the Constraints bullet states the cause without claiming how the removal left the ref.
- F5 (MEDIUM), fixed as proposed: D8 covers an adopted local warp branch, read by the warp probe that the adopt-only option runs, and drops its ref and upstream config; new D8 criterion.
- F6 (MEDIUM), fixed as proposed: D2's way forward before Land names `lym batten cancel <slug> --remove` and what it clears.
  The first D2 criterion, D3's two mentions and the D3 criterion use the same form.
- F7 (MEDIUM), fixed as proposed, taking the first option: under the adopt-only option `Add` reuses a records worktree registered on the pair's weft branch and makes only the task worktree.
  D1's Bound and D2's records-worktree answer now cover only a directory that is not the pair's.
  That answer's way forward clears the directory and re-runs at any row, replacing the `lym fabric remove` route.
  New D1 criterion for a resume whose task worktree alone was deleted.
- F8 (LOW), fixed differently: the not-live bullet defines the candidates, the by-SHA fetch writing no branch or tag, the `done` filter and the latest-committer-date pick.
  I did not filter by the run id the batten status records: the self-check found that the status records none (`internal/shedengine/status.go`).
  The bullet instead states why the latest date picks this run's tag.
- F9 (LOW), fixed as proposed, taking the first option: the not-live bullet names running `lym batten run <slug>` right after the restore and states the cleanup window.
- F10 (LOW), fixed as proposed: D1 says the entry is judged from origin's Bolt and the batten run record, and that the parent's branch is judged only by the second reconcile.
- F11 (LOW), fixed as proposed: D3's owned-elsewhere clause deleted.
- F12 (LOW), fixed as proposed: the Constraints bullet is cut to the settled cause and its evidence source, and the timeline moved to the support log's Round 2 section.
- F13 (NIT), fixed as proposed: D6 says an archive tag of D4's covers the lease tip, at it or at a descendant.
- F14 (NIT), fixed as proposed: D1 step 2 runs `git worktree prune` in both repositories.
- F15 (NIT), fixed as proposed: Open risks reads "None."; the Constraints bullet's push-both-sides clause is deleted, and so are the first and third notes.

## Self-check

- fix-consistency, MEDIUM, records-branch half of `refuseAdoptLocalCheckedOut` missing from D2's closed set: fixed as proposed (merged with fix-way-forward's same finding); the closed set names both refusals, and the at-any-row way forward covers both.
- fix-consistency, LOW, D8 Bound silent on the upstream config: fixed as proposed; the Bound says a rollback restores neither.
- fix-consistency, LOW, Scope bullets lag D7 and D8: fixed as proposed.
- fix-consistency, LOW, Q5 in the question ledger names a bare cancel: fixed as proposed.
- fix-claims, MEDIUM, the status records no run id: fixed as proposed; the run-id filter is dropped, and the latest-date pick is justified on its own.
- fix-proposal, MEDIUM, `Add`'s rollback would remove a reused records worktree: fixed as proposed; D1 says the rollback leaves it in place.
- fix-proposal, MEDIUM, the warp probe's answer would make `Add` fetch the task branch over a stale ref: fixed as proposed; D8 says under the adopt-only option the answer serves only the drop.
- fix-proposal, LOW, `--unset-upstream` fails on a branch with no upstream: fixed as proposed; D8 and D2 unset only an upstream that is configured.
- fix-proposal, LOW, a `commit-records` commit after D4's read is given up: fixed as proposed; D7's Bound states it.
- fix-way-forward, MEDIUM, the Land restore names a weft `git branch` over a live weft: fixed as proposed; the bullet names the tag restore only when the weft is not live, and the task-branch restore only when that branch is gone.
- fix-way-forward, MEDIUM, records branch checked out elsewhere is transient: merged into the fix-consistency finding above.
- fix-way-forward, LOW, the checked-out refusal names `lym fabric add --adopt-local`: fixed as proposed; the stop names `lym batten run <slug>` in place of it.

## Disputed

None.

## Deferred

None.
