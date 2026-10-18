# Card 16 — batten-start-after-numbers

**Edit:**
- `docs/research/batten-start-2026-10-18/README.md`
- `internal/battencli#newCommitStatusSeam`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#battenCLI.commandDeps`
- `internal/shedadapters#CommandProducer.Call`
- `internal/shedrecipe#commandEntry`
- `internal/battencli#TestBattenIntegration_Rows`

**Uses:**
- `internal/battencli#wireForHub`

**Intent:** Take D2's after numbers, cut the per-transition overhead they show in batten's own path, and pin the start's cost in the integration-tier drive.

Measure exactly as card 1 did, now over cards 2 to 15, and fill the report's `After` section with per-row numbers beside the before numbers, naming again the rows measured live and those measured only in the drive.
When the implementer cannot take a live batten start, the report says so and that the orch adds the live after-numbers for Open-Window and Seed-IDE on the first batten start after landing (the decision record's open risk).
A cost the measurement finds in batten's per-transition path beyond the status commit and the row's own action is removed in this card, within the members D2 names that path by: the seam and its marker (`newCommitStatusSeam`, and `battenCommitStatusDeps` with its seed stat), the config reads (`commandDeps`, whose lookups may share one snapshot per row reach), and the producer wrapping (`commandEntry` and `CommandProducer.Call`).
A cost the measurement finds anywhere else is outside that path and is recorded in the report with where it lies.

`TestBattenIntegration_Rows` gains a start-cost step, on its own slug, with the board default layer's commands set as the step needs: a row whose switch is off, and an `Open-Window` whose command only starts a background process (`sleep 0 &`-style).
The step reads each row's cost from the status history's transition timestamps, and subtracts the status commit's time, which it measures by wrapping the wired `CommitStatus` seam in a timer, and the process's start, which the command's own marker records; it reads no trace, since a test binary arms no trace sink.
It asserts each of the two rows costs under 100 ms beyond those, the decision record's D2 acceptance criterion, which is itself a wall-clock bound; the step carries a `//testtiming:keep` reason naming that criterion and runs at the integration tier only.

**ImpactSummary:** Only batten's own seam, its Command-row wiring and the Command producer's entry and call can change, and only by removing cost; the report gains its `After` section.

**Commit:** 16: Research: batten start after numbers, and the overhead they show cut
