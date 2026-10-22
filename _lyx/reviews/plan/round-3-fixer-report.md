# Round 3 fixer-report — plan

## Changes

- F1 (MEDIUM, consistency): fixed as proposed in `_lyx/plan/04-board-owner-writes.md`, `TestSetOwners` paragraph.
  "a diverged pull refusing naming its detail" became two rows, each naming its input.
  Diverged: a local `notes/c.md` commit against a second clone's pushed `notes/c.md`, so the replay conflicts and the setter refuses naming `SkipDetail` and writes nothing, with a note that a `board.json` conflict cannot be the input.
  Dirty: an uncommitted edit to a tracked `notes/x.md` while a second clone pushes a change to it, so the fast-forward is blocked and the setter refuses naming the dirty detail, which names `lyx board sync`.
  Premise checked against `internal/fabricengine/bolt.go:239-259`, `bolt.go:308` and `internal/fabricengine/boltpullfirst_integration_test.go:225-259`.
- F2 (MEDIUM, consistency): fixed as proposed in the same paragraph.
  Added a push-side transport row: `BeforePush` repoints origin to `http://127.0.0.1:1/board.git` after the pull's fetch has succeeded, and `SetOwners` returns `ErrOwnerOriginUnreachable` wrapped with the push's detail, keeping the commit local and leaving the slug out of `Recorded`.
  The existing unreachable row now says it reaches the pull's `fetch_failed` skip.
  Premise checked against `classifyFailure` in `internal/fabricengine/pushclassified.go:234`.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
