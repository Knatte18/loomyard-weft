# Round 2 fixer report

All four findings fixed; `approved:` in `00-overview.md` untouched.

## Changed

- F1 (`05-rerun-removes-superseded-strand.md`): the strand teardown stays on `Attach` and `AttachGated`, the entries callers use when they start a run on a not-found answer, and a new `Runner.AttachIfLive` is the same probe with the teardown off.
  The two Bouncer wait-only probes (`awaitLiveJudge`, `awaitLiveSeed`) move to it; the other Bouncer probes, the reflection probe, SingleLLM and Burler each start a run on not-found and keep the teardown.
  The card gains `bouncer.go`, `singlellm.go` (the `Shuttle` seam) and `shedfake.go` as targets, a bound sentence, and a test row asserting `AttachIfLive` removes nothing; the earlier paragraph that extended the teardown to probe-only callers is gone.
- F2 (`13-orch-reload-sequence.md`): the plugins case in `typeReloadStep` is stated to persist the move itself, skip the shared `confirm` tail and never call `advanceReload`; the restart test row also asserts nothing else is typed on that tick.
- F3 (`11-watcher-reloads-fresh-compaction-only.md`): `initCursor` clears `w.compactedAt` on a strand change, stated in the held-boundary paragraph, with a test row for a strand change between a holding tick and a tick that reads no turn end.
- F4 (`10-compaction-boundary-turn-ends.md`): `ReadTurnEndAfter` is false when the payload carries no message, with the consequence stated and a `usage_test.go` row added.

## Gate repair

The gate blocked Card 5 on `handle-unreferenced`: its `plan:internal/shuttleengine#Runner.AttachIfLive` Create handle was used by no other card.
`AttachIfLive` is now described in the card's prose and edited in through `attach.go`, with no Create handle.
The gate's `create-new-unit` finding on Card 18's `validate.go` is informational and the new file is intended, so nothing changed there.
The review file is unchanged: F1's finding and fix are the same.

## Deferred

None.
