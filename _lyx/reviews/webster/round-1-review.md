---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomrecipe/resume_test.go:317
    summary: test comment still explains the Bouncer exhausting first by a one-ahead Stuck sequence under equal budgets, which card 1's exempt hand-backs make false
---

Range: no `_lyx/loom/rework/` round exists, so the review range is `git diff 292bec755..HEAD`, the merge base with `product.parent` `main`.
Six commits, one per card, plus one fix commit.
`lyx gate test` over `internal/shuttleengine`, `internal/battenshed`, `internal/shedadapters`, `internal/loomrecipe` and `internal/shedengine` passes.

### [LOW] Stale one-ahead rationale in the budget-exhaustion test comment

**Location:** `internal/loomrecipe/resume_test.go:317`
**Issue:** The comment on `TestBounceRouting_BudgetExhaustionBlocks` says the Bouncer exhausts first because its Stuck sequence runs one ahead of the round producer's count, "with equal budgets".
Card 1 makes both Burler hand-backs `BudgetExempt`, so the Burler row's count is not compared with the Bouncer's at all; the Bouncer's check is the segment's only budget on judged rounds.
Card 1 rewrote the same claim in `internal/shedadapters/burler.go` and `bouncer.go` but left this copy.
The test's assertions are unaffected.
**Fix:** Reword the last paragraph to say the Bouncer is the row that binds because the Burler's hand-backs are exempt, and drop the one-ahead and equal-budgets reasoning.

## Judgments on the focus questions

- Cards 5 and 6: the guard (`armHoldGuard`, `pollEventsTick`) and the hold's end (`turnStartRead`) read turn starts only through `turnStartOffsets` over the tick's own `data` and `startOffset`, via `SessionSignalParser`, with no read of `waitshadow.go`'s fold. The only `waitshadow.go` change is the `stands` field. Not a defect.
- Card 4: `sentAt` is read once, after the load turns and immediately before `sendVerified`. It sets `run.offset`, `PromptOffset` and the late-landing check's reference. `TestStartup_SkillLoadTurns` has a hook-appended line row that would fail on the old code. Not a defect.
- Card 3: `Awake` defaults to a monotonic closure in `NewInnerRun`, and `InnerRun.Awake` is in the `envkit` nil-legal list. `observeClocks` runs before the unreadable-activity early return and drops the accumulation only on a stamp that moves after a recorded one. `sleepGap` clamps a negative excess to zero. Tests use the fake clock only.
- Card 1: both Burler hand-backs are exempt and the gate-failed exit stays counted. `roundBudgetExempt` and its test are gone with no remaining callers. The Bouncer's code is unchanged, and `escalation_test.go` was corrected to count every hand-back as exempt.
- Card 2: loom and darn headers and `docs/overview.md` state the episode rule. No "per-task budget" or "five rejection rounds" wording remains.
- Tests: every new case extends an existing test except `TestSleepGap`, which the shared decision permits. Top-level counts of `shedadapters` and `shuttleengine` dropped or stayed, and `battenshed` is at 87 against its row, so `test-budget.yaml` is rightly untouched.
