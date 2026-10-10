# Fixer report: discussion round 1

Every finding was checked against the code and fixed.

## Fixed

- **F1** (BLOCKING): confirmed, because `internal/stencilcli/cli.go` commits `lyx stencil sync` output as `lyx: seed stencils`, the pre-run's message.
  D3 now gives sync its own prefix, `lyx: sync <subtree>`.
  D5's signature matches only the pre-run's `lyx: seed ` prefix and excludes sync and hub-wide config commits.
  D5's Bound now lists what a seed commit can hold (seeded, refreshed and restamped-to-default files, plus `.gitattributes`), never a hand edit or a forced refresh, and says Bolt holds the pulled copy until a refreshing binary runs.
  Two acceptance bullets cover this: sync's prefix and the primitive refusing a sync commit.
- **F2** (BLOCKING): confirmed.
  D5's primitive now resets to upstream and replays the non-seed ahead commits in order, so the board commit that triggered the push survives.
  On a replay conflict it restores the original tip and refuses.
  The Bound, the Rationale, D4's divergence rule and the Bolt push acceptance bullet now say what lands.
  Both rejected options are recorded in the support log.
- **F3** (BLOCKING): confirmed.
  `cmd/lyx/stencilseed.go` clears `sourceDir` in a consumer repo and passes `stencilstore.Source{}` to the specs pass.
  D1 now records each writer's commit time beside its revision.
  Ancestry decides when the worktree holds both commits, and a strictly later commit time decides when it does not.
  The specs pass gets the same answer through its `Source`.
  D1 also gained a Bound: only production builds from pushed main refresh, so only clock skew can invert the order.
  Scope, Constraints (`buildvcs`), Open risks (skew replaces the task-worktree risk), the Acceptance criteria (fallback and specs) and the Notes (`vcs.time`) were updated to match.
- **F4** (MEDIUM): confirmed.
  D5 no longer claims D4 covers the config commit.
  D4 now says an unpushed hub-wide config commit hits the divergence rule.
  When it does, the stamp stays absent, every start verb warns naming `git pull --rebase` in the board dir, and due pre-run writes skip until the operator acts.
  The Open risks entry and an acceptance bullet match.
- **F5** (MEDIUM): confirmed, because `HeadContains` (`internal/gitrepo/blobread.go`) runs only through go-git, and `IsAncestor` (`internal/gitrepo/ancestry.go`) is CLI-bound.
  D1 now names a presence check through `readGoGit` followed by the `merge-base --is-ancestor` reachability read through `gitexec`.
  The Constraints line and the Notes match.
- **F6** (LOW): D9's Decision now says what gc may newly delete (unreachable loose objects between one day and two weeks old) and that only the expiry bounds it.
  The D8 claim is gone from its Rationale.
- **F7** (LOW): under D8, a listing with no tmux server now reports no sessions.
  Any other probe error skips gc with a Warn carrying the error.
  The Bound, the Open risks entry and an acceptance bullet cover the always-failing probe.
- **F8** (LOW): new acceptance scenarios cover the heal's fetch, drop and fast-forward, the config commit never being dropped, the failing probe, and the zero `Mode`.
- **F9** (LOW): the Scope Out item now states the fact without "Part 4".
  D4's Rationale says "a clone behind its upstream", and the 1829-commit incident moved to support-log Interview batch 4.

The support log also gained the round's auto-picks in the Question ledger and the rejected options under Rejected alternatives.

## Disputed

None.

## Deferred

None.
