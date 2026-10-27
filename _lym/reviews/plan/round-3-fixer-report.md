# Fixer report — plan, round 3

## Changes

All in `_lym/plan/08-dobby-worktree-gone.md`.

- F1 (LOW, consistency): fixed as proposed.
  The after-tick-read paragraph now logs a non-decode read error through a `logger.Episode` of its own, a new `stateRead` field of `daemon` beside `ownerRead`, as `beamOwnedElsewhere` logs an unreadable owner, instead of through `d.failures`.
  "keeps the earlier `keptTidy` and goes on" is kept.
  The proposed "a read that does not error ends that episode with `Recover`" is worded "a read that does not fail with such an error", so a decode failure, which `ReadJSONNoMkdir` returns as `ErrDecode`, also ends the episode rather than leaving it open.
  `internal/logger#Episode` is added to the card's **Uses**.
- F2 (LOW, scope): fixed as proposed.
  `TestRun_WorktreeGoneExits` gains a row after the directory-replaced `state.json` row: tick 1's hook overwrites `state.json` with bytes that are not JSON while `.git` stays present, the `Deps.Sleep` wrapper removes the worktree, the record reads as absent, `keptTidy` becomes nil, and the fake records no `StopStrand`.
  The paragraph's sentence on a failed decode now names that row.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
