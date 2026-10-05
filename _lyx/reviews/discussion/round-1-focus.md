---
round: 1
focus:
  - "D3, open-map rule: inside a declared open map, key typos are no longer reported by Reconcile, MissingKeys or Set. Does the Decision state what that lets through, and is the stated bound (declared paths only, scalar-only Set) enough, given D1's names-only validation?"
  - "D2, legacy list tolerance, together with A2 and R1/R2: is the decode-only bound consistent with what D3's Reconcile carry does to a list-shaped value when the template is a map? Settle whether a list-shaped file survives `lyx config reconcile --apply` and `--set labels.x=...` without a shape clash, or the Decisions leave that undefined."
  - "D7, unconditional issue carry: the bound says a merge only adds issue numbers. Does the Decision say what happens when carried numbers collide with intake's one-entry-per-issue uniqueness rule or with validateWrite's issue validation, or is that left unbounded?"
  - "D9 and D10, removed picker and early refusal: what can the refusal before cwd resolution now let through (a module named like a subcommand, a name that is both), and does the Decision bound it? Is removing the interactive picker (A1) in scope per Scope, or an unrequested removal?"
  - "D5, `get --body`: verbatim output with no envelope and exit 1 on an absent target. Does the Decision say how a caller tells a missing entry from an empty body, given stdout is empty in the empty-body case and an envelope goes to the error path in the other?"
  - "Consistency between Scope, Decisions and Acceptance criteria: each Scope 'In' item has a Decision and a criterion, each criterion traces to a Decision, and no Decision (D4 `board labels`, D11, D13) is missing from Scope. Check the D4 'reads board.yaml only' claim against R4 and D3's reach into configengine."
  - "Relocation candidates: does `decision-record.md` carry deliberation narrative that belongs in `support-log.md` (long Rationale lines, Open risks R3 on burler's maps, Out-of-scope reasoning)? Before any move, apply the completeness-before-leanness test: extract any requirement the plan writer needs into Decisions or Constraints first."
  - "False premises: the Scope and Out claims about current behavior (the `board notes` exceptions removed in `ee1ffa4b`, `cliTreeFindings` holding exactly the three config/selfreport entries, live `board.yaml` files being lists) are stated as fact. Check them against the repo and git log, at the level of whether the premise holds, not exhaustive enumeration."
---

Skipped per the rubric's Do-not-flag list: the optional Notes section, rejected alternatives in the decision record, and call-site enumeration.
Skipped per the writer stencil's symmetry rule: exact signatures, file paths and `file:line` citations are not asked for, so their absence or imprecision is not a focus.
