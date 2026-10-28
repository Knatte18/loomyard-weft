# Fixer report — plan, round 1

## Changes

- **F1** (BLOCKING, fixed as proposed): `11-report-replies.md` gains an `**Extends:**` group holding `internal/reportengine#allowedImports` before `**Uses:**`, and an Intent sentence: `allowedImports` gains `internal/creelengine` and `internal/agentname`, the two packages `ComputeReplies` imports, so `TestToldGeometryInvariant_AllowlistOnly` passes.
- **F2** (BLOCKING, fixed as proposed): `07-chase-clock.md` creates `chaseProbeCeiling = 8 * time.Minute`.
  The clock paragraphs now say: one `agedPersistInterval` wait while the clock runs; a tick that reads it stopped doubles the next wait up to the ceiling; a running tick or a wake of the holder returns it to the base; time is charged only between two running ticks one interval apart, so a backed-off wait charges no stopped time; a wake during a backed-off wait starts a fresh base wait.
  `isChaseWait` matches `agedPersistInterval <= d <= chaseProbeCeiling`.
  The PATTERN-daemon-wakeups chase-timer bullet states the backoff.
  `TestScheduler_Chases` gains the dormant-holder backoff row.
  The overview's `chase-clock-cadence` Decision and Rationale are reworded to match.
- **F3** (MEDIUM, fixed as proposed): `05-expect-reply-engine.md` adds `Store.settledOutcomeLocked` to Edit and an Intent bullet: its reply-wait case answers a non-EXTERNAL sender's wait with the oldest reply whose `ReplyTo` is the id, journaling no `delivered` record since the reply stays mail; card 7's unanswered case comes after it.
  `TestStore_Mail` gains the cancelled-`PostWait`-then-`WaitReply` row.
- **F4** (MEDIUM, fixed as proposed): `07-chase-clock.md`'s Journal-and-fold list gains a bullet: `applyPosted` leaves `programs` alone for every `posted` record whose sender's program name, read with `agentname.ParseProgram`, is `ChaserProgram`, a chase and a sender notice alike.
  The chase-only exemption clause is removed, `agentname#ParseProgram` joins Uses, and `TestJournal_Fold` gains the sender-notice row.
- **F6** (MEDIUM, fixed as proposed): `06-expect-reply-cli.md` gains the Bound paragraph.
- **F7** (LOW, fixed as proposed): `05-expect-reply-engine.md` gains a `**Uses:**` group holding `internal/loomcli#noticeFits`.
  The Intent and the ImpactSummary now say that `noticeFits` measures a decision notice against the marked form, a conservative bound.
- **F8** (LOW, fixed as proposed): `09-sweep-holder-gone.md` adds `pattern/PATTERN-daemon-wakeups.md` to Edit and the sentence on the sweep bullet's widened run condition.
- **F9** (LOW, fixed as proposed): `12-stencils-through-creel.md` gains an `**Edit:**` group holding `copyTestdataStencil` and `TestSmokeSeatsChairAsksAdvisor`.
  It also gains the sentence on the smoke test's header and its comment above the advisor's history polling, and the `**ImpactSummary:**` line.
- **F10** (LOW, fixed as proposed): `07-chase-clock.md` plans `readers` as a map from program to the journal day of its read.
  `carryRecords` writes a carried read record for each reader before the cutoff, `applyCarried` folds it back, and `moveOrigins` moves the day.
  `TestJournal_Retention`'s row asserts that a program whose only read is past retention stays in `readers`.
- **F11** (LOW, fixed as proposed): `05-expect-reply-engine.md` drops the two `TestStore_Mail` rows asserting `Replied` and keeps the `ReplyTo` log-entry assertion.
  The `TestJournal_Fold` row extends to a forwarded marked message replied to by its new holder, asserting `Replied`.
- **F12** (LOW, fixed as proposed): `02-creel-post-parent.md` keeps one resolution row, from beam `b` with its orch live, beside the three refusal rows.
  The dropped walks are attributed to `TestAsker` (card 1) and `internal/orchcli#TestParentOrchStrand`.

F5 is `decision`-class and is left out of this report.

## Self-check

- fix-claims: no findings.
- fix-affected-set, LOW: `sweepNeeded`'s and `sweepLoop`'s doc comments in `internal/creelengine/sweep.go` keep the old sweep run condition. Fixed as proposed: card 9 adds both to Edit (moving `sweepLoop` from Uses), and its Intent says the doc comments of `sweepNeeded`, `sweepSubjects` and `sweepLoop` name the widened condition.
- fix-consistency, LOW: card 10's `creelcli#isChaseWait` still described only minute waits. Fixed as proposed: card 10 now has it match one minute to eight minutes, the values of card 7's unexported constants, and notes that its row keeps the holder live.
- fix-proposal, LOW: a holder coming back live fires no wake, so a stopped clock's restart goes unseen and uncharged for up to `chaseProbeCeiling` plus one interval. Fixed as proposed: card 7's clock paragraph and the overview's `chase-clock-cadence` Rationale state this lag.

## Disputed

None.

## Deferred

None.
