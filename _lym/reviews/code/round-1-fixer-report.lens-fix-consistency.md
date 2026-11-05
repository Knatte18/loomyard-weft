```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-set through the fix-consistency lens, looking for twins and siblings the changed text left stale.

- **LOW** — `contracts/stencils/loom/loom-template-discussion.md:107`, "A question only the run's parent can settle is not settled from your own exploration…".
  - The fix reworded the same claim at `bouncer-fragment-decision.md:7`, `shedadapters/doc.go:108`, `circling.go:300` and `shed-recipe-spec.md:61`.
  - Loom's Discussion-Review rows set `escalate_decisions`, so a decision finding now reaches the arbiter first, and the arbiter settles it from the board entry and decision record.
  - This line is addressed to the discussion agent, whose own rule (settle nothing from its own exploration) still holds, so it is not clearly wrong.
  - A reader would still notice that it keeps the old "only the parent" framing beside the reworded twins.
  - It is stale only if the claim was meant as the general definition of a `decision` finding.

- **LOW** — `loom-template-arbiter.md`, `circling` bullet after the edit.
  - It now reads "…then continue below the hard cap. At the hard cap accept only when the finding is wrong or the fix in place already answers it. Otherwise hand on."
  - The first sentence's "Otherwise hand on" case (the board entry and record do not settle the finding) now sits after the at-cap sentence.
  - A reader can take "Otherwise" as covering only the at-cap sentence.
  - The meaning still works, but the three branches (settled below the cap, at the cap, not settled) are no longer separated cleanly.

No other stale twins found. The remaining grep hits are unrelated (`carry it out` in shuttle tests, `loom-rubric-code-review.md`, and the bench prompt copies under `docs/research`). `loom-rubric-discussion-review.md:66` was already rejected by the reviewer.

The code twins agree:
- `doc.go` says the outcome line is sent after the consume and not on a refusal.
- `arbitrate`, `noticeArbiterSettled` and the `goDriver` field comment say the same.
- The test row names match the behaviour.

Tool calls: 1 (`grep -rn` for the old phrases over `.go`, `.md` and `.yaml` outside `_lym` and `.lym`). My context holds only the files the fixer read, so it could not show other files carrying the same claims.
