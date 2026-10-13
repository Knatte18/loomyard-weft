# Card 4 — env-hard-cap

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/testkit/envkit#nilLegal`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`

**Intent:** Each Bouncer row receives its perch's resolved hard cap from the run's Env (D3), so the loom and darn wiring (cards 6 and 7) have one field to fill.

`Env` gains `RowHardMaxBounces map[string]int`, placed after `RowAutoAccept`: the resolved hard cap of each Bouncer row, keyed by row name as `RowAutoAccept` is.
A nil map, a missing entry and 0 tell the row no hard cap.
Unlike `RowMaxBounces`, its values are already resolved, since the per-perch fallback lives in the caller's config.
`bouncerEntry` passes `HardCap: env.RowHardMaxBounces[name]` into the `shedadapters.BouncerConfig` it builds.

`nilLegal` gains `"RowHardMaxBounces"`: a nil map is the documented no-hard-cap default, so `NilSeams` over `envkit`'s full Env does not report it.

`TestBouncerEntry_BlockingEscalatesAndColorSegment`'s table gains a `hardCap int` field, set as `env.RowHardMaxBounces = map[string]int{"review-bounce": tt.hardCap}` when non-zero, and a new row "a hard cap of 1 offers only accept".
That row runs `blocking_escalates` with hard cap 1 and wants `Awaiting`, with a Reason holding `lyx loom circling accept` and no `lyx loom circling continue`.
The row captures the Call's pointer for that check.
The round-1 judge writes an open BLOCKING entry and the row's budget is spent and auto-accepted, so before this card `BlockingEscalates` escalates with cause budget and a Reason naming both verbs, and the row fails if `HardCap` is not passed through.
Once card 5 lands the same round reaches the rule's hard-cap step and still awaits with the accept-only Reason.

**ImpactSummary:** One new optional Env map and one BouncerConfig field filled from it; every existing Env builder leaves it nil, which keeps today's behavior, and only the recipe tests' nil-seam allowlist needs it named.

**Commit:** 4: each Bouncer row takes its perch's hard cap from the Env
