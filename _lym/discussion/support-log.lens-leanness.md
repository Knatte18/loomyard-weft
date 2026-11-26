```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content to move or cut in the decision record and support log.

1. MEDIUM — Acceptance criteria, "Review: the raddle split follows D9's order…" (D9). A code review reads the diff a part leaves, and no part's diff contains the raddle split, so no reviewer can check this bullet. Raddle's `order_departures` key already holds a departure from an order the record states. Fix: drop the bullet, or rewrite it as behaviour a part's diff shows (for example, each part changes only its own concern's files). Before cutting, give D9 its own coverage so `decision-not-in-acceptance` cannot fire. Today D9 appears in no other bullet.

2. LOW — Goal. The parenthetical citing `fabric-test-cuts-remeasure-2026-11-08.md` ("348 tests dropped bought 10 percent") argues why trimming was rejected. That deliberation is already in the support log's Rejected alternatives ("Trim or fold more tests"). The Goal only needs the cause. Move it out, or leave the citation only under Constraints.

3. LOW — Constraints, "Baseline facts". This restates the Goal's "211 s of a 250 s tier1 run", so the number appears twice. The shards figure repeats the support log's rejected alternative, and the plan writer gains nothing from it. Keep the baseline once, in D10, which is the decision that uses it. Cut it from Goal or Constraints, and drop the shards sentence from the record.

4. LOW — Auto-mode assumptions. Each bullet restates its decision's content ("two packages, not one, so a rule change…", "the 60 s target"). The section is required, so keep it, but cut each line to the bare pick and its id, for example "Concern order (D9)", so the decision's substance lives only in the decision.

5. LOW — Constraints, the PATTERN bullets. `PATTERN-leaf-packages` ("both new packages are leaf packages…") repeats D1, and `PATTERN-test-only-code` repeats D8. The fabric-boundary bullet's "the new packages inherit… (see `## PATTERN changes`)" repeats that section. Keep only what the decisions do not already say.

6. LOW — D1 and D12 give illustrative lists ("such as the add and sync options…", "such as `internal/fabriccli`, `internal/shedengine`…"). Both are hedged with "such as", but the D12 list is the parent's file tally in another form and will go stale as the moves land. D12's rule ("a test that pins that module's behaviour moves to that module") stands without it. Cut it to "another module's package", or keep it as a pointer only.

7. LOW — Support log, Interview Turn 2. Each item lists its alternatives, and the Rejected alternatives section then gives the same alternatives with reasons. Turn 2 should keep only the pick and point to Rejected alternatives.

Other (outside leanness):

8. MEDIUM — Open risks, third bullet. "Each concern's part finds such reads while moving it" leaves the question of which mid-flow re-reads the snapshot cannot capture for implementers to find out later. This is close to a deferred research question, though it may fall under the plan-level call-site fence. Either reword it as a design rule (any re-read the snapshot lacks becomes a snapshot field or an action-list boundary, per D3) or keep it as a stated risk. Either way, no step should be told to discover the fact.

Tool calls: none; my context held everything this lens needed.
