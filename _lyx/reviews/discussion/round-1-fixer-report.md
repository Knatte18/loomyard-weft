# Fixer report — Discussion-Review round 1

All six findings fixed; `lyx loom validate-discussion` exits 0 after the edits.

## Changed

- **F1 (MEDIUM)** — `decision-record.md` Decision 4 no longer claims a per-run bound of `Attempts` re-prompts.
  It states that a capped `PassOnCap` entry never runs again, that a pass before capping resets the count and allows a fresh streak, and that a request which must not repeat makes its closure idempotent.
  The rationale now says "a capped request".
  An acceptance scenario now covers the reset for `PassOnCap` entries too.
- **F2 (MEDIUM)** — Decision 6 defines `Passed` as true only when every non-off entry without `PassOnCap` passed at the final arrival, so a failed or not-reached check entry makes it false.
  Decision 2 now defines `finalize`'s arrival: memo if Wait left one, fresh evaluation otherwise, same ordering and cap rules, never re-prompts or changes a count.
  `FindingsPath` names the entry that failed at the final arrival.
  A new acceptance scenario covers the not-reached case.
- **F3 (MEDIUM)** — The first acceptance criterion now says existing tests keep their assertions, gaining an explicit `Attempts` of 3 where they relied on the removed default, and shedrecipe's tests of the removed keys are replaced by the `gates` tests.
- **F4 (LOW)** — The Refusal Way-Forward constraint and the fifth auto-mode assumption now settle that the new shedrecipe construction errors are wiring guards (recipes are embedded), with no refusal-spec rows added.
  Support-log ledger item 12 is resolved accordingly.
- **F5 (LOW)** — The Out paragraph drops the review gate's closure design and keeps the per-run limit, now stated as content for the `PassOnCap` doc comment.
- **F6 (NIT)** — Decision 1 drops the nil-test rewrite clause.
- `support-log.md` gains a `### Discussion-Review round 1` entry under Review rounds and ledger items 15 and 16 for the two changed decisions.

## Deferred

None.
