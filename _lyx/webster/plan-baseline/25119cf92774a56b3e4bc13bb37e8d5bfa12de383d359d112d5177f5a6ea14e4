# Card 17 — batten-done-refusal

**Edit:**
- `internal/battencli#battenCLI.doneSlugRefusal`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#TestRunInWindow`
- `internal/battencli#TestBattenPreStep_RefusalKinds`

**Uses:**
- `plan:internal/battencli#battenCLI.cleanupCmd`

**Intent:**
The done-slug refusal names `lyx batten cleanup --apply` for the run directory instead of telling the operator to delete it as a change on the pair's fabric sibling, and keeps its advice on the leftover task branch (D9).
The text reads: `battencli: %q has already completed; to run it again, run "lyx batten cleanup --apply" from the prime, which removes its run directory %s, and delete the task branch its torn-down pair left behind, locally and on the remote ("lyx fabric cleanup --apply --remote" removes any orphaned sibling branch), or give the new task a different slug`.
The run verb's `Long` in `battenVerbTexts`, "A slug already done refuses on the envelope, naming the per-slug directory and the leftover branches to delete to run it again", says the refusal names `lyx batten cleanup --apply` and the leftover branches.

A grep for the old wording, `delete its run directory`, finds its exact-string assertions in `TestRunInWindow` and `TestBattenPreStep_RefusalKinds`; both assert `lyx batten cleanup --apply` in its place and keep their other substrings, which the new text still carries.

**ImpactSummary:** One refusal's text and the run verb's help change; the two tests asserting the old text are on this card.
