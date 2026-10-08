---
round: 2
focus:
  - "Card 9 (`_lyx/plan/09-resume-circling.md`) refusals in `loomcli`'s resume path: every refusal it adds or changes for an awaiting run, including a recipe lookup error and a decision file read error from `PendingCirclingDecision`, needs a `refusal-spec.md` row in the same card, a `resume_test.go` row that reaches it, and a way forward that fits that cause (a malformed decision file does not heal on a bare re-run of the verb). Settle whether each such refusal has all three."
  - "Cards 3 and 7 add the `shedrecipe.Env` fields `CarryOver` (a func) and `RowReviewModels` (a map). `battencli`'s `TestWire_EverySeamFilled` fails for any nil func or map field of `Env` that batten neither fills nor lists in its `intentionallyNil`, and `envkit` has a `nilLegal` list for legal nils. Settle whether the owning card lists the file or entry that keeps that test green at its commit."
  - "Card 9 changes the way forward of the circling verbs to `lyx loom resume`. Settle whether every other place that states the resume step after a circling decision is in the card's edit list, namely the loom root command's `Long` text in `internal/loomcli/cli.go` and the review-segment escalation paragraph of `docs/overview.md`."
  - "Card 1 (`_lyx/plan/01-carry-over-write.md`) result check after the carry-over write: the decision record's D2 bound requires re-reading the new file and checking that its H2 heading sequence equals the prior file's, that each segment has at most one well-formed marker pair, and that every line outside the replaced span (entry lines, or appended or removed lines) equals the prior file's. Settle whether the card's check covers all three, including the append and removal cases."
  - "Card 6 (`_lyx/plan/06-review-budget-three.md`) edit to `internal/loomrecipe/escalation_test.go`: settle whether any claim the card makes about what that edit pins is true of what the test reads, given the test sets its own review budget constant and the template value is read in `config_test.go`."
---

Round 2 re-checks the new text each fix introduces, and the rest of the rubric applies as usual.
