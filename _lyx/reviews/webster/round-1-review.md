---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shedadapters/bouncer.go:688
    summary: A malformed escalation record halts the Bouncer with a Reason that names no way forward, and the new refusal has no refusal-spec row and no reaching test
---

Range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, no rework round exists).
`go build ./...`, `go vet ./...` and the unit tests of every touched package pass.

Judged clean against the focus directive:

- Cards 9 and 10: `unearnedCircling` and `isGatingEntry` use the Shared Decisions' gating definition, the checkpoint comes from `loom.yaml` through `Env.ReviewCirclingCheckpoint`, and `settleUndecided` picks cause `budget` whenever `budgetReached` holds, including on a `CIRCLING` verdict.
- Cards 4 and 11: `loomrecipe.New` and `Routing` apply `review_max_bounces` to every row of a Bouncer segment, `segmentBounces` feeds the same value to the seam, and `budgetReached` uses the same `count >= budget` as Shed.
  `roundBudgetExempt` marks the round producer's Stuck exempt only after a budget `continue`.
- Cards 7, 8 and 10: the escalation files, the `cause` frontmatter and the legacy `circling` reading match the Shared Decisions.
- Card 15 and card 3 mechanical checks: a repo-wide grep for the removed symbols finds only `internal/treadleengine`'s unrelated `LedgerEntry`; `ledgeraccess.go` and its test are gone; the overview says four anomalies; the two config keys appear in the struct, template, validation and overview.
- Comment conventions: the new doc comments use one sentence per line and pin no tally.

### [BLOCKING] Unreadable escalation record is an unspecified, untested halt

**Location:** `internal/shedadapters/bouncer.go:688`
**Issue:** `escalate` returns `Stuck` with the Reason "escalation record for round N is unreadable: ..." when `readEscalation` fails.
This is a refusal new in card 10.
Its message names no way forward, `contracts/specs/refusal-spec.md` has no row for it, and no untagged test reaches it.
`PATTERN-refusal-way-forward` and the plan's Shared Decisions both require the way forward, the row and the reaching test in the card that adds the refusal.
**Fix:** Name the way forward in the Reason (inspect or remove the named escalation file, then run `lyx loom start`), add a `refusal-spec.md` row for it, and add an untagged test in `bouncer_escalation_test.go` that seeds a malformed record and asserts the halt and its Reason.
