```yaml
round: 1
fixed: [F1, F2, F3, F4]
disputed: []
deferred: []
```

## Changes

- **F1** (fixed differently, commit `7a43773d`): the proposal asked the `Notify` seam to resolve the orch strand's name at each attempt.
  It already does: `orchcli.NotifyPrime` calls `PrimeOrchStrand` inside every `postNotice`, so each retry in `sendNotice` re-resolves the receiver.
  The fixed `<shortname>:orch` answer for a GUID reed no longer holds is already pinned by the row "a GUID the reed state does not hold takes the fixed orch name" in `internal/orchcli/notify_integration_test.go`.
  A test of that name at the `hubDeps` seam cannot run: creel's daemon is not spawned under `go test`, so a refused post fails before any receiver is named (seen in the tier 1 run).
  Made instead: the seam in `internal/hibernatecli/deps.go` logs at Warn when `NotifyPrime` returns posted false with a nil error, and `internal/hibernatecli/deps_integration_test.go` pins that over a hub whose prime records no orch.
  Whether the retry alone answers the board entry's second bullet, that is, whether the first failure's receiver `ly:orch` was the unlisted fixed name or a listed one, is not decidable from the code; the parent records it with `lym loom decision add` if the retry is meant as the whole answer.
- **F2** (as proposed, commit `5167fc3d`): two `TestWatch` rows in `internal/hibernateengine/watch_test.go`, a short multi-line notice routed to the file and a notice-file write failure naming itself in the short line.
- **F3** (as proposed, commit `5167fc3d`): `Watch` in `internal/hibernateengine/watch.go` builds `target` before `reconcileAtWake` and passes it, so the daemon's own entry is recognised when the request records no `Owned`; one `TestWatch` row clears `Owned` after the hibernate and asserts the reconcile ran, the entry is lifted and the notice does not say a pause stands.
- **F4** (as proposed, commit `5167fc3d`): two `TestWatch` rows, a `finish(PhaseDone)` write failure after the lift, and a fail notice with a long reason written whole to the notice file.
- Self-check finding, doc sentence on a notice dropped for want of an orch strand, added to `internal/hibernateengine/doc.go`.
- Gate repair: the doc sentence was first written without its `//` prefix, so `doc.go` no longer parsed and every package importing the engine failed to set up; the prefix is added.

The engine's tier 0 and the hibernatecli tier 1 gates passed before the self-check and the doc sentence.

## Self-check

- fix-consistency, LOW: the `# Notices` section of `internal/hibernateengine/doc.go` did not cover a dropped notice; fixed with one sentence.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
