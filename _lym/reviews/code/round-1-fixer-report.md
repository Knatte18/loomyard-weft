## Changes

- **F1** (LOW, fixed as proposed): `stepChases` and `releaseGoneHolder` read the hub's strands through the new `listStrands` before taking the store mutex and pass the reading to `settleUnansweredLocked` and `noticeUnansweredLocked`, which no longer call `Strands()` under the lock.
  A failed listing surfaces only in the notice that needs it, so chases and settles proceed.
  Commit `2816a54f`.
- **F2** (LOW, fixed as proposed): `internal/orchcli/notifierwiring_test.go` holds one table test over the burlercli, loomcli and webstercli wirings, asserting each `runner.SetNotifier` closure calls `orchcli.NotifyAsker`.
  The shared-test form is the one the review's `fix` allows.
  Commit `a832f396`.
- **F3** (NIT, fixed as proposed): the clause naming the removed shuttle send is gone from the creelengine package doc.
  Commit `7f5461ec`.

The verify gate then failed `TestTierTags_OneTagPerTestFile`: the new F2 test file named no tier tag.
It now carries `//go:build tier0`, and `lym gate test ./internal/orchcli ./cmd/lym` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW: the strands are read on every `stepChases` and `releaseGoneHolder` call, not only when a settle is due. Disposition: fix as proposed.
  The lazier shape needs a two-pass lock re-entry for a cost the reviewer's own proposal accepts, and the listing sits outside the mutex, which is the premise of F1.
- fix-proposal, LOW: the F2 test scans the wiring source instead of running the notifier. Disposition: fix as proposed.
  The runner's notifier is private and built inside the engine stack, so a behavioral test needs a hub with a creel daemon in three packages; the review allows one shared wiring test, and the repo already has source-scan tests.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
