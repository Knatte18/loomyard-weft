# Card 4 — verdict-vocabulary

**Edit:**
- `internal/shedadapters/bouncerfiles.go`
- `internal/shedadapters/bouncerfiles_test.go`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/doc.go`
- `internal/shedadapters/bouncer_clear_test.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedadapters/bouncer_commit_test.go`
- `internal/shedadapters/bouncer_replay_test.go`
- `internal/shedadapters/attachprobe_test.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/focus_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomrecipe/sequence_test.go`
- `internal/loomrecipe/resume_test.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `docs/overview.md`

**Create:**
- `internal/shedadapters/bouncer_verdict_test.go`

**Intent:** Replace the judge's APPROVED/BLOCKING verdict with `CONVERGED`, `CONTINUE` and `CIRCLING` (D1), and map `CIRCLING` to an Awaiting halt.

`bouncerfiles.go`:
the verdict constants become `verdictConverged`, `verdictContinue` and `verdictCircling`.
`parseVerdict` accepts exactly those three words and rejects everything else, legacy words included.
A new `parseRecordedVerdict` reads a verdict that was already on disk at Call entry: it tries `parseVerdict`, and otherwise reads a legacy `APPROVED` as `verdictConverged` and a legacy `BLOCKING` as `verdictContinue`, reporting that the word was legacy.
`recordedVerdict` uses `parseRecordedVerdict`, so the clear, the replay and BurlerProducer's may-advance check all read an old binary's run directory as judged.
A new `harvestedVerdict(runDir, round)` reports the strict verdict only: legacy words never count as a harvest.

`bouncer.go`:
the two harvest sites — the entry-time attach and `judgeCall`'s post-run check — settle only when `harvestedVerdict` parses.
When the files hold a legacy word instead, the Bouncer archives the three judge outputs through `archiveStaleOutputs` and degrades with a Reason saying the judge wrote a retired verdict word, so `judged(n)` reads false and the round is re-judged under the new prompt.
The Call-entry clear fires on `verdictConverged`, including an aliased `APPROVED`.
`settle` maps `verdictConverged` to Approve, Commit and Done with the ledger pointer, and `verdictContinue` to `ensureFocus(round+1)` and Stuck with the ledger pointer, as the old two branches did.
`verdictCircling` returns `shedengine.Awaiting` with the ledger pointer and a Reason naming the segment, the round and that the judge found no progress; it calls `ensureFocus(round+1)` and spawns nothing.
Because Awaiting re-calls the same producer, a re-call over an on-disk `CIRCLING` verdict reaches the replay and returns the same Awaiting without spawning;
the existing entry-time probe still attaches a live judge first.
Awaiting is a returned verdict, so it survives cancellation like the other two.
Rewrite every doc comment in `bouncer.go` that names APPROVED or BLOCKING, including `NewBouncer`'s budget rule and `Call`'s pointer rule.

`doc.go`: rewrite the Outcome-mapping section's opening (the Bouncer now produces Awaiting) and its Bouncer bullet around the three verdicts and the legacy aliasing, stating the bound: a legacy word never settles a harvest.

`bouncer-template-judge.md`: the verdict file section asks for exactly `CONVERGED`, `CONTINUE` or `CIRCLING`, and every APPROVED mention becomes `CONVERGED`.
The reading list and role text are rewritten in card 6.

Tests in `bouncer_verdict_test.go`:
`parseVerdict` accepts each new word and rejects `APPROVED`, `BLOCKING` and a lowercase `converged`;
an on-disk legacy `APPROVED` at Call entry clears and re-seeds, and a legacy `BLOCKING` replays as Stuck to the round producer;
a judge spawned in this Call that writes `APPROVED` returns no Done, leaves `judged` false and archives the three outputs;
an attached judge that writes `APPROVED` is treated the same way;
`CONVERGED` runs Approve then Commit and returns Done with the ledger pointer, and the next Call clears and re-seeds;
`CONTINUE` returns Stuck with the ledger pointer;
`CIRCLING` returns Awaiting with the ledger pointer, and a second Call returns Awaiting with no shuttle Run;
a second Call with a live judge still attached on a `CIRCLING` round attaches and harvests rather than respawning or archiving.

Every existing fake judge and fixture in the listed test files that writes APPROVED or BLOCKING writes `CONVERGED` or `CONTINUE` instead, and comments naming the old words are updated.

`docs/overview.md`'s spawn walk-through says the Bouncer returns `Done` on a `CONVERGED` verdict.

**ImpactSummary:** Changes the Bouncer's verdict contract for all three loom review segments; every test fake that scripts a judge verdict across shedadapters, shedrecipe and loomrecipe switches words, and old run directories stay readable through the entry-time aliases.

**Commit:** 4: bouncer: verdicts are CONVERGED, CONTINUE or CIRCLING
