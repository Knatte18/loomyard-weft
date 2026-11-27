```yaml
lens: walkthrough
outcome: reported
```

Task: walk each decision of the self-healing-runs decision record through one concrete case, end to end, and check the end state against its purpose.

**Findings**

1. **BLOCKING, D6 and D9 against D10: a run the orch holds on purpose gets chased, then posted as severe.**
   - Case: under `unattended`, a `budget` escalation still has an open BLOCKING finding the fixer did not defer. The orch pauses the run with a note and posts `[operator] held <slug>:` (D6).
   - A pause is not a transition. D10 ends a chase only on "a resume, a goto or an accept", so after `beam.halt_chase_min` the run is still "halted at the same stop with `settle: orch`".
   - D10 chases it, and one interval later posts it as a severe halt ("no settling action open"), which mentions the operator (D8).
   - D9's paused-instead-of-cancelled run takes the same path. So the two holds meant to stay quiet overnight wake the operator. D10's "wait by design" does not cover a held or paused run.

2. **BLOCKING, D2 and D3: routine review endings become severe.**
   - Case: Discussion-Review escalates `decision` in round 1 and again in round 2, and the arbiter resumes both. D3 counts an arbiter resume as a retry, so the count is 2.
   - Round 3 hits `hard-cap`. That is the third halt at the row, so D2's hard-cap row returns `operator` instead of the arbiter accepting.
   - The peer `push-retry-not-halt` went through exactly this sequence: escalations, then hard-cap in round 3, accepted by the arbiter. Under this record that run pages the operator for a mechanical accept, which defeats "only truly severe".
   - The count needs to be per review segment, or exclude escalations that end with an arbiter resume or accept.

3. **MEDIUM, D2 table, board claim row: the third stop has two verdicts.**
   - lym settles twice (counts 1 and 2). The orch column says "from the third stop" and the severe column says "third halt at the row", so count 3 is both `orch` and `operator`.
   - The arbiter rows have the same gap: they never say what count 2 returns (arbiter again, or orch).

4. **MEDIUM, D3: the dedup key is not stated.**
   - "Each stop counted once, whether it left a halt entry, a settle record or both" gives no key that links a status-history halt entry to a go-settle record.
   - Case: the tackler resumes a Stuck. That writes one halt entry and one settle record, and nothing in the record says they are the same stop. The count is likely 2 where it should be 1.

5. **MEDIUM, D6: the arbiter can accept a red gate.**
   - `settleUndecided` also escalates `budget` when the live gate is red (`liveRedDetail`), not only for open BLOCKING findings.
   - Case: every open finding is marked `deferred` but the gate is red. D6's rule accepts the round, which lands a red gate. D6 does not handle the red-gate cause.

6. **MEDIUM, D4 against D5: the pause wait holds the driver lock, so resume can never succeed.**
   - Case: a bench pauses the run, and the driver now waits holding its driver lock instead of parking.
   - The operator or bench runs `lym loom resume`. D5 waits for a park that never comes while the pause stands, then refuses with `driver_not_parked`, naming a retry that cannot succeed.
   - The record also says nothing about how a driver kept alive by the pause wait interacts with a hub pause or hibernate meant to stop sessions.

7. **MEDIUM, D8 and D10: the dobby has no input for the verdict.**
   - D8's reminders of "a run held at a severe halt" and D10's "run handed back with `settle: orch`" both need the verdict.
   - D7 puts the verdict only into the creel notice stamps and the driver's settle record. The dobby reads child status, pause and the state record, never the notices.
   - No step hands the verdict to a record the dobby reads.

8. **MEDIUM, D7 and Open risks: severe posts route by program name, not by the run's parent.**
   - `internal/slackengine/outbound.go` routes a program post by its name: `<shortname>:<NAME>` goes to the hub orch, `<shortname>:<slug>:<NAME>` to that beam's orch.
   - The Open risk says D7's posts "follow the run's recorded parent", but D7 names no program identity derived from that parent.
   - Case: a beam member's driver posts as a hub program, so the post goes to the hub orch's channel, and is dropped if that orch's button is off. D12 then warns about the wrong orch.

9. **LOW, D10: "by D7's path" names a path that does not run.**
   - The unattended severe post after a chase is made by the dobby, but D7's path is the driver's hand-back, and the driver is parked by then.

10. **LOW, D7: the post may be skipped when no orch strand is recorded.**
    - `handBackRun` queues notices only when an orch strand is recorded. If the Slack post is added beside them, a run with no recorded orch never reaches Slack.
    - D7 does not say the post is independent of that condition.

11. **LOW, D1: a mistyped level reads as `false`, not as a refusal.**
    - `hubgeom.Autonomy` "reads false on a failed read", so a value such as `unatended` silently means no autonomy at run time. The refusal D1 promises shows only in the lookup's own callers.

12. **LOW, D8: severe halts can notify twice.**
    - The orch posts a relay to `<shortname>:SLACK`, and the hook also forwards its turn reply opening `[operator] severe halt`. Both carry the mention, so the operator gets two notifications for one halt.

**Walk results per decision**

| Decision | Case walked | Purpose met? |
|---|---|---|
| D1 | `unattended` set in a board `run_settings` | Yes, apart from finding 11 |
| D2 | board claim row at counts 1 to 3 | Ambiguous (finding 3) |
| D3 | tackler-resumed Stuck; review rounds 1 to 3 | No (findings 2, 4) |
| D4 | bounce budget exhausted twice | Yes |
| D4 | pause, then resume | No (finding 6) |
| D5 | resume during a network wait | Refuses with a dead-end way forward |
| D6 | deferred-only round with a red gate | No (finding 5) |
| D6 | held run under `unattended` | No (finding 1) |
| D7 | beam member's severe halt | Misrouted (finding 8) |
| D8 | split ready line | Yes, quiet |
| D8 | severe-halt reminder | No input (finding 7) |
| D9 | cancel replaced by a pause | No (finding 1) |
| D10 | decision recorded, run not resumed | Yes |
| D10 | paused run | No (finding 1) |
| D11 | agent awaiting a creel reply | Yes |
| D12 | button off under `unattended` | Yes, but the reached orch is wrong for beam runs (finding 8) |
| D13 | glossary lines | Yes |

**Extra tool calls**
- Grep of `internal/sheddriver/drivehandback.go` and `drive.go`: checked when a hand-back notifies and whether it parks. My context had only a summary.
- Grep of `handBackRun` and listing of `internal/dobbyengine` watch files: checked the orch-strand condition and what the run watch reads. My context did not have it.
