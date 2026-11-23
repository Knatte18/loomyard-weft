# Card 8 — restarts-start-pusher

**Edit:**
- `internal/battencli#DobbyRuns`
- `internal/beamcli#hubDobbyDeps`
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.wire`
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/battencli#TestBattenPreRun_ContinuePass`
- `internal/loomcli#TestResumeVerb`
- `internal/beamcli/doc.go`

**Create:**
- `plan:internal/battencli#TestBattenPreRun_ResumeStartsThePusher` -> `func TestBattenPreRun_ResumeStartsThePusher(t *testing.T)`

**Uses:**
- `internal/runpusher#Start`
- `internal/runpusher#StartIfAhead`
- `internal/runpusher#TargetOf`
- `internal/runpusher#RestartPusher`
- `internal/battencli#newFakeReceiver`
- `internal/loomcli/doc.go`

**Intent:**
Restart a pusher on every path D5 names besides the next commit: the dobby's tick and a resume.

Every restart goes through `runpusher.RestartPusher` (card 5), so a failed start only warns.

- `DobbyRuns` calls `RestartPusher` with `runpusher.StartIfAhead` on the pair's target
  inside the existing `!inputs.OwnedElsewhere && view.PairPresent` condition, beside the code-tip settle.
  Bound: a pair owned elsewhere gets no pusher from the tick;
  the catch-up's `HandOverOwnedElsewhere` pushes that run's owed commits itself before it hands the run over.
  The run stays in the tick's views whatever the start does.
  The read of ahead counts a pair whose records branch is level and whose code branch holds its records tip's code tip origin lacks,
  so this tick carries the restart of a code push the owner dobby's code-tip settle no longer makes (D6).
- `hubDobbyDeps`'s prime `Tick` calls `RestartPusher` with `runpusher.StartIfAhead` on the prime's target before its other work.
  Only the hub-prime dobby fills `Tick`, so one dobby per hub restarts the prime's pusher.
  `internal/beamcli/doc.go`'s dobby-seam section names the prime's pusher restart among the prime dobby's hub-wide duties, at the start of its `Tick`.
- `battenCLI` gains `startPusher func()`; a nil seam starts nothing, so a hand-built receiver reads nothing.
  `battenCLI.wire` fills it: it starts the prime's pusher and, when the run's pair stands here, the pair's with the run's slug,
  each through `RestartPusher` and `runpusher.Start`.
  `battenPreRun` calls it on a resume, a found status with no `--continue-pass`, after the owner pre-flight
  and before the `noReopen` return and the reopening, so the pusher starts before the run acts with or without `--no-reopen`.
  A batten resume re-arms its loom child with `lym loom start`, so this call is also the pair's resume start.
  A continue pass, which is no resume, starts none.
- `loomCLI` gains the same `startPusher func()` seam, nil starting nothing; `loomCLI.wire` fills it over the pair's target with the run's slug.
  `resumeFromRunState` calls it once it has read a found status that is not done, before any driver wake.

`TestBattenPreRun_ContinuePass` is tier0 and asserts a continue pass calls no `startPusher`.
`TestBattenPreRun_ResumeStartsThePusher` is tier0 on a receiver from `newFakeReceiver` with a found status and a recording `startPusher`:
a resume calls it once before the reopening, and a `--no-reopen` resume calls it once too.
`TestResumeVerb` is tier0 and gains rows with a recording `startPusher` that notes the order against the driver wake:
a paused run's resume calls it once, before the wake; a done run's calls none.

**ImpactSummary:** Pusher restarts in the dobby's run read, the prime dobby's tick and the two resume paths, filled in both wires; a failed start only warns.
