```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens review of `_lyx/plan` (overview plus cards 1–18).

**Findings**

1. **MEDIUM** — Card 16, D11 "carrying in-flight runs across": a run halted at `Merge-In` is not treated as a hand-back until a loom verb has renamed its status.
   - Card 16 changes `internal/landingshed#HandsBack` to match only `parentAlignName`.
   - The status rename (`renameRetiredRows`) runs only in `internal/loomcli#loomCLI.armAt`, so only loom verbs reach it.
   - Batten reads the child's status file directly and calls the predicate without arm: `internal/battencli/wire.go:934` (`arbiterstop.HoldReading(taskLocation, status)`, which reaches `internal/arbiterstop/arbiterstop.go:107`) and `internal/battencli/wire.go:975`.
   - A run halted spent or over-cap on `Merge-In` at upgrade therefore reads as no landing halt there. Batten's eligible-stop hold then treats it as arbiter-eligible instead of handing it back.
   - The old row name is guarded on every loom path but not on this batten path.
   - Fix: `HandsBack` also matches `retiredMergeInName`, plus a `TestHandsBack` row for a `Merge-In` status.

2. **LOW** — Card 12, Routing: the card says a settled accept "is never reached here, since `settledGeneration` takes it". Two `settledGeneration` branches still send a settled `decision` accept back into `settleUnlessRefused`, where `routeDecisions` has no branch for it.
   - `entryHarvest` calls `settleUnlessRefused` directly (`internal/shedadapters/bouncer.go:441`).
   - `entryReplay` falls through to `bouncer.go:494-497`. Card 12 makes `standsAsRecorded` false for a `decision` cause, so that path reaches `settleUnlessRefused` too.
   - The states are narrow:
     - `entryReplay` needs a settle record that is not live, meaning a generation mismatch (`internal/shedadapters/settlerecord.go:182`).
     - `entryHarvest` needs a generation record but no settle record. On the Discussion segment `beginSettle` writes the settle record before `settleCirclingAccept`, so this state is hard to produce.
   - Fix: `routeDecisions` returns handled false over a settled accept, so the existing settle path runs.

3. **LOW** — Card 12, test row 12: a `continue` "recorded" at the hard cap on a `decision` escalation cannot be made through the verb.
   - `circlingVerb` refuses `continue` once the latest round is at or past the hard cap and an escalation is pending (`internal/loomcli/circling.go`, the `hardCap &gt; 0 &amp;&amp; latest &gt;= hardCap` branch).
   - So the re-escalation is reachable only when a continue was recorded below the cap and the cap was lowered before settle, or under an amendment generation's smaller cap.
   - The row writes the decision file directly, so it pins a branch no entry point reaches at a fixed cap. The card's Intent should name the reaching input, a hard cap lowered after the record.

4. **LOW** — Card 12, Routing: `routeDecisions`' guard only steps aside for an escalation record of another cause, not for a decision file of another cause.
   - Example: the soft-cap rule's automatic `continue` or `accept` (cause `budget`, `automatic: true`), which writes a decision file with no escalation record (`internal/shedadapters/circling.go`, `recordAutoAccept` and `recordAutoContinue`).
   - With the switch on, a round written by an older binary that holds such a file and a `decision`-class finding would be read as a pending parent answer to a `decision` escalation.
   - Reachable only across the upgrade. Fix: also hand back when the decision file's cause is not `decision`.

**Per guard**

- Card 3 unknown-gate refusal: reachable.
- Card 4 no-`review`-group: reachable.
- Card 12:
  - routing ahead of the refusal (harvest, attach, replay): reachable.
  - `settledGeneration` false on an unsettled `decision` round: reachable.
  - None and pending-continue branches: reachable.
  - pending-accept branches (rejudge, retire and re-escalate, hard-cap `Halt`): reachable.
  - settled-accept branch: reachable, unplanned (finding 2).
  - hard-cap continue: reachable only via a cap change (finding 3).
- Card 13 switches: reachable.
- Card 14 arbiter refusal: reachable once card 12 makes the cause valid.
- Card 15 rename migration: reachable.
- Card 16 `readAttempts` fallback, fixer-report fallback, status rename: reachable.
- Card 16 `HandsBack`: unguarded on batten's direct status read (finding 1).

**Tool calls**

- `internal/shedadapters/settlerecord.go` `liveSettleRecord`, and the `settleUnlessRefused` call sites in `bouncer.go`: needed what makes a settle record live, which the exploration did not hold.
- The rest of `liveSettleRecord`: needed the later-review clause that ends liveness.
- `internal/arbiterstop/arbiterstop.go` around `HoldReading` and `LandingHalt`: needed how the hold derives the landing-halt bit.
- Callers of `arbiterstop` and `landingshed.HandsBack`: needed which read a status without loom's arm.
