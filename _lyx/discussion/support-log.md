# Support log — plan-gaps

## Interview

Autonomous mode: no operator answered; every decision below is an auto-pick recorded in the ledger.

1. Read the board entry `plan-gaps` and the entries touching it: `fixer-critical-of-findings` and `burler-multi-llm` both edit the burler fixer stencils, and `plan-multi-llm` is a later Plan-Write restructure.
   Conclusion: keep this task's changes out of the burler stencils and reach the Plan-Burler fixer through the Plan-Review rubric.
2. Explored the plan format (`contracts/specs/loom-plan-spec.md`): card numbers tie together file name, heading, Card Index line, default and explicit commit subjects, `first_card` and webster's begun/done record.
   Conclusion: one card insert legitimately touches every later card, so the fix is a verb that does it mechanically, not a looser file rule.
3. Explored the Plan-Burler fix scope (`fixScopeRules`, overlay: "exactly the target paths"), the edit directive (Edit or Write only), and the agents' standing denies (no `rm` deny).
   The target path is the whole `_lyx/plan` directory and `CommitPlan` commits the whole directory, so deletions already reach the commit.
4. Explored the frozen-card rule: the rubric's write surface, `done-card-edited` and `lyx webster rebaseline`'s "at or before the last begun batch's last card" refusal.
   Conclusion: the verbs refuse positions in the begun region.
5. For the refusal gap, read `PATTERN-refusal-way-forward` (a new refusal is reached by a test), `refusal-spec.md`'s wiring-guard class, and the code-quality rule against code paths the code's guarantees prevent.
   Conclusion: the plan writer names each prescribed refusal's reaching input and test, and Plan-Review flags a card that cannot.

## Rejected alternatives

- **A rule allowing `rm` inside the fixer's write surface (issue #508's second option).**
  It sanctions deletion but leaves the twelve-file renumber as a hand rewrite, which is the error-prone part.
- **Card files named by slug alone, with order taken from the Card Index.**
  Removes renaming but not the heading and commit-prefix rewrites, and changes the plan format, webster's resume trail and every check keyed on numbers; far larger than the problem.
- **A general `lyx loom renumber` verb that compacts any numbering.**
  Repairs an inconsistent plan by guessing intent; the precondition refusal plus the gate's own findings already name the inconsistency.
- **Rewriting prose mentions of moved card numbers.**
  A mention may mean the old or the new card; reporting them is safe, rewriting is not.
- **Changing `burler-step-3-fix.md` or the fix-scope rules to name the verbs.**
  Those are generic across segments and are being edited by `fixer-critical-of-findings` and `burler-multi-llm`; the plan rubric reaches the Plan-Burler fixer already.
- **A mechanical reachability check in Go.**
  Reachability depends on what upstream code rejects; no plan-time parser can decide it.
- **Adding a Webster-Review rubric rule against constructor-only refusal tests.**
  The board entry asks for a plan-time fix; the plan rubric bullet already states that a constructor-only test does not reach a refusal.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): verb or file-removal rule for card insertion? Verb pair `insert-card`/`remove-card`; it removes the bulk rewrite, not only the `rm`.
- Q2 (resolved, auto-pick): verb placement? Flat `lyx loom` subcommands beside `validate-plan`; format rewrite in `planparser` beside `SetApproved`.
- Q3 (resolved, auto-pick): what the inserted card holds? Heading only, Card Index intent from a required `--intent` flag; the gate catches an unfilled stub.
- Q4 (resolved, auto-pick): may the verbs shift begun cards? No; positions at or before the last begun card are refused, matching rebaseline.
- Q5 (resolved, auto-pick): prose mentions of card numbers? Reported in the envelope, never rewritten.
- Q6 (resolved, auto-pick): how agents learn of the verbs? One sentence each in the plan and rework stencils and the plan rubric's write surface; burler stencils unchanged.
- Q7 (resolved, auto-pick): reachability check mechanical or review? Review: writer rule plus a matching rubric bullet.
- Q8 (resolved, auto-pick): Webster-Review rubric in scope? No; recorded as rejected.
- Q9 (open, for the plan): does `lyx webster rebaseline` require each renumbered unbegun card to be named with `--card`? Recorded under Open risks.
