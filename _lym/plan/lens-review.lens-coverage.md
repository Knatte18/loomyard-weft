```yaml
lens: coverage
outcome: reported
```

Coverage lens: every decision, acceptance criterion and new branch in the four cards, matched to the card and planned test row that would fail if it broke.

**Findings**

1. **MEDIUM** — AC2 / D1 ("resolves the conflict with the conflict session under the parent lock"). No planned row asserts the conflict session runs while the parent lock is held.
   - Card 1's `TestFinalize_MergeJudgement` "conflict session resolved" row uses a struct-literal `Finalize` with a zero `guard`, which takes no lock (`internal/landingshed/finalize.go:114-116`).
   - The `TestFinalize_OverRealHub` first step only asserts one session and Done.
   - The "under the lock" half of the criterion has no row. A row could check the lock record from inside the session fake.

2. **MEDIUM** — AC8 (batten's binary re-step leaves a halted escalate stop alone). `TestRestepSkips` pins `restepSkips`, but no row pins that `battenCLI.wire` passes `restepSkips` as `InnerRun.HandBack`.
   - If `wire` kept the old `arbiterstop.HandsBack` closure (`internal/battencli/wire.go:1066-1068`), every planned test would still pass.

3. **LOW** — AC7 (a judged-first stop hands back when the arbiter is off or its budget is spent). Card 3 says these hold through existing rows instead of planning one.
   - The existing arbiter-off row drives an awaiting escalation (`internal/sheddriver/drive_test.go`, the row named "awaiting stop… arbiter_max_spawns: 0 turns the arbiter off").
   - No row covers a blocked stop that reached `driveActionArbiter` through `ArbiterFirst`. The action path is the same, so the risk is small.

4. **LOW** — D3 (the escalate halt text "still names the row, the report path and the conflict"). Card 2's `TestFixerRun_Outcomes` true-flag fragments are the report path, `decision D3`, `lym loom decision add` and `lym loom resume`. None of them is the row (`fixerTestRow`), so dropping the row from the reason passes.

5. **LOW** — Card 1's `TestFinalize_MergeJudgement` conflict row. Once `conflictSessions` and the `session` field are removed, the "merge the conflict session resolved" row drives the same fake input as the "non-overlapping parent change" row.
   - Its `eventResolver` returns resolved and not up to date, and starts no session.
   - So the row repeats a covered behaviour, and the conflict path is reached only by the tier1 integration test.

6. **LOW** — Operand check, Card 4 `loomCLI.specFor` (`c.recipe != shedrun.RecipePawl`). The record says "loom and darn tell the escalate predicate". The card's condition also tells it for an empty recipe, which a hand-populated receiver has.
   - Card 2's `landingDeps` uses the same inverse form.
   - It is harmless for the three recipes, but it is not the operand the record names.

**Tool calls**
- `sed -n 585,640p internal/sheddriver/drive.go`: read the driver's start check and open-attempt reuse, to judge whether AC7's start-path claims hold through existing rows. My context had only the `facts`/`recheck` excerpts.
