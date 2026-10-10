# Fixer report: discussion, round 2

## Fixed

- F1 (MEDIUM, design): D1 now says `ForceRefresh` records the running binary's revision and commit time only when D2 lets that binary refresh (production channel, a revision, an unmodified tree).
  From any other binary it writes the body with no recorded revision, so the next production binary refreshes the copy as an unstamped one.
  Added an acceptance criterion pinning both cases.
  Validated against `internal/stencilcli/cli.go:158`, where `syncCmd` calls `ForceRefresh` with the running binary's `Source` whatever its channel.
- F2 (LOW, consistency): D5's signature now requires D3's parenthesised label after `lyx: seed <subtree>`, and states that a pre-change binary's seed or sync commit, which carries a bare `lyx: seed <subtree>` (`internal/stencilcli/cli.go:165`, `:187`; `cmd/lyx/stencilseed.go:150`), never matches and fails the push as today.
  Added an Open risks bullet for that case and extended the drop-refusal acceptance criterion to a commit lacking the label.
- F3 (LOW, consistency): D4 now separates the two failures: a transport failure clears at the next board push, and a rebase conflict clears only when the operator resolves `git pull --rebase` in the board dir, since `reconcileHubWide` pushes through the same `bolt.Push` (`internal/hubreconcile/walk.go:201`) and D5 never drops a hub-wide commit.
  The Open risks bullet no longer names a board push as an exit for the conflict case.
- `support-log.md`'s Question ledger records the three round-2 picks.

## Disputed

None.

## Deferred

None.
