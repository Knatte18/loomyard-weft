```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already touch what D5 to D8 introduce.

**Paths traced**
- D7 to the gate key decoder in `internal/gateslot/config.go` (`decodeGates`). A `reports` field decodes, since `node.Decode` is not strict and only `lints` names are validated. The step survives, but nothing validates its names (see finding 4).
- D7 to the lint step site in `internal/gatecli/run.go`. That step runs after the tiers only when the call names a gate key, the context is alive and the run did not halt. A halted or cancelled call writes no report. D8's freshness check covers that case, so the step survives.
- D7 to the green record carry (`green.go`). A carried tier run still reaches the lint step, so the report step runs on a carried call too. It survives.
- D7 to the shipped key tiers in `internal/gateslot/template.yaml`. The `webster`, `darn-write` and `code-burler-fix` keys run only `[0]`, and the template declares no tiers. It breaks for judging `tier1` tests (finding 2).
- D6 to `countLandings` and the tidy settle in `internal/dobbyengine/tidy.go`. `LandedSinceTidy` is cleared by tidy settle, counting pauses while a tidy runs, and the first read is a baseline that counts nothing. It breaks if the step reuses that list as its trigger (finding 3).
- D8 to when the report is written and when HEAD moves. The writer's gate runs before the row commits, and Webster-Burler-Fix commits after the `webster` gate. It breaks (finding 1).
- D5 and D7 to the `gateslot` process cap, watchdog and run log. The test binaries the step runs sit outside any tier run, and the record says nothing about them. Unverified (finding 5).

**Findings**

1. **BLOCKING, D7/D8: the commit freshness key turns the report stale on every normal path.**
   - The step records HEAD's commit while it judges the diff from the merge base, uncommitted work included.
   - The `darn-write` gate runs before Darn-Write commits, so the reviewed HEAD is a later commit and D8 shows "no current report".
   - On motif, Webster-Burler-Fix commits after the `webster` end gate, so its review gets no report either.
   - The green record in `internal/gatecli/green.go` keys on HEAD's tree and treats a dirty worktree specially. That is the model to follow: key the report on the tree the step judged, and decide what a dirty worktree records.

2. **MEDIUM, D5/D7: running a `tier1` test under "its tier's `env`" with the gate's prebuilt `lym` rests on things the shipped keys do not provide.**
   - `internal/gateslot/template.yaml` declares no tiers, and the three keys D7 ships on run `tiers: [0]`.
   - A hub that does not declare `tier1` has no env to give the step, and a key that runs only `tier0` builds no prebuilt `lym`.
   - The record should say how the step gets a `tier1` env and a built `lym` when the key does not run that tier, or that it reports such tests as not judged.

3. **MEDIUM, D6: "reusing the landing count the board-tidy step already keeps" breaks against how `countLandings` works.**
   - The tidy settle clears `LandedSinceTidy`.
   - Nothing is counted while a tidy agent is in flight.
   - The first read baselines without counting, so a fresh dobby never triggers a first build.
   - The map step needs its own pending marker in the dobby state record, set by the same fresh-landing detection. The record should also say whether a fresh dobby starts a first refresh.

4. **LOW, D7: `decodeGates` refuses an unknown lint name and should refuse an unknown report name the same way.**
   - Without that, `reports: [redundncy]` is silently ignored.
   - The refusal needs a way forward per `PATTERN-refusal-way-forward`, which the record does not list.

5. **LOW, D5/D7: the step's and the verb's test processes run outside a tier run.**
   - The record does not say whether `runProcessCap`, the watchdog and the gateslot run log apply to them.
   - A hung test in the step could stall a gate call that D7 says never fails, since the record gives no step timeout.

**Extra tool calls**
- Grepped `internal/gateslot/config.go`, `green.go` and `rungate.go`: my context did not hold how gate keys decode, how carry works or where lints run.
- Read `decodeGates` and the call sites of `Lints`: to confirm lint validation and when the step can run.
- Read the lint-step block in `internal/gatecli/run.go` and `countLandings` in `tidy.go`: to trace the halt path and the landing-count semantics D6 reuses.
