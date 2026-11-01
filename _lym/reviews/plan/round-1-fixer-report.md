# Round 1 fixer-report — plan

## Changes

- **F1** (fixed as proposed): card 16 drops `PartsMax` from **Env**, the wiring, `envkit.FullEnv` and the rejection rows.
  The `parts` gate requires a non-nil `Setting` and reads `env.Setting("tapestry", "parts_max")`.
  It refuses a value that is not an integer of 1 or more, naming `tapestry.parts_max`.
  The rejection rows are now a nil `Setting` and a `Setting` answering 0.
  The seam counts read four.
- **F4** (fixed differently): card 12 drops the committed golden, the `TestPlanTable` edit and the "holds no `Step `" assertion, and keeps the marker rows.
  Not made: the proposed comparison "as line multisets".
  The parent's added decision requires Plan-Write's prompt to stay the same line for line, in today's order, so the one-time check compares the before and after prompts line for line, with nothing committed.
- **F5** (fixed as proposed): card 12 edits `loom-plan-spec.md`, `webster-spec.md`, `docs/overview.md` and the plan-review rubric's pointer.
  Each pointer now names the plan-format blocks that `loom-template-plan.md` includes.
- **F6** (fixed as proposed): card 7 extends `contracts/specs/loom-plan-spec.md`.
  The spec's sentence listing the context-check modes (at line 414; the review cited 416) gains `--part`, the `plan-part` gate and the writer's Accept.
  The validation checks gain a `part-export-missing` row.
- **F7** (fixed as proposed): card 17 moves `internal/reportengine/doc.go` to **Edit**, and its plan-phase list names both new rows.
- **F8** (fixed as proposed): card 5 moves `internal/planglyph/doc.go` to **Edit**, and the doc states both part-scope exemptions.
- **F9** (fixed as proposed): card 9 rewords the `doc.go` sentences on the lead include, the up line and the step end.
  It also rewords the `Seat.Required`/`Seat.Accept` doc comments and the `Table.Validate` doc comment's lead-directive rule.
- **F10** (fixed as proposed, then corrected by the self-check): chair Step 4 gains a bullet that rewrites each `Applies to` line to the joined card numbers.
  The numbers are read by slug from the Card Index after every spot card is inserted.
- **F11** (fixed as proposed, on the branch where `part-export-missing` stays): card 19 item 3 names a failing part carrying `part-export-missing` for an unmet export.
  Card 7's `JoinParts` paragraph names `planparser.JoinParts`' refusal as the backstop that card 6's `TestJoinParts` reaches.
- **F12** (fixed as proposed): card 18's `TestValidatePlanCmd_Part` drops the `part-spot-target` row and keeps the dangling-handle row as the one failing row.
- **F13** (fixed as proposed; reaching test adjusted by the self-check): with `--part` set, `validatePlanCmd` refuses unless `loomrecipe.HasEngine(recipe, "PartsPlanWrite")`, using the proposed text.
  No run reaches the refusal today, so the card says it guards a later recipe.
  `TestValidatePlanCmd_Part` reaches it by building the CLI with the darn recipe and running the command without arming it, as `TestValidatePlanCmd` does.

### Carried for the parent's recorded decision (F2, F3)

These two are decision-class findings.
They are listed here only because the parent's added decision ("Plan-Write's prompt stays line for line; parts file is never edited by the chair; part check flags a missing export") instructed these changes:
- Card 12 splits the moved text into four blocks, each included where its text stands today: `loom-block-plan-glyphs`, `loom-block-plan-format`, `loom-block-plan-overview` and `loom-block-plan-cards`.
  The three lines that do not fit every including stencil stay in `loom-template-plan.md`.
  Card 14's writer and chair stencils include the same four blocks and state their own versions of those lines.
  The overview's Shared Decision and card index line are rewritten to match.
- Card 6 adds `planparser.ExportMet`.
  It matches a `plan:` export by the declaration it names, meaning the same `ParseRef` `Unit` and `Name`, so the canonical spelling still meets the parts file's spelling.
  `JoinParts` and `ValidatePart` both use it.
  Card 7 drops the re-parse and the respell way forward, and keeps `part-export-missing` as D7's fourth difference.
  Card 14 drops the chair's parts-file edit, says the chair never changes the parts file, and leaves the writer one finding to report.

## Self-check

- fix-proposal, MEDIUM — F10's printed numbers go stale after `insert-card`: fixed as proposed (the bullet reads numbers from the Card Index by slug after the spots are inserted).
- fix-affected-set, LOW — card 9 left other advisor-only comments: fixed as proposed.
  Card 9 rewords the `Table.Validate`, `validateSeatRoles`, `Seats` and `chairDirectiveBlock` comments and `doc.go`'s "every later seat is an advisor", and adds `internal/seatengine#chairDirectiveBlock` to **Edit**.
- fix-affected-set, LOW — the spec's `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted` rows omit `ValidatePart`: fixed as proposed (card 7's spec sentence adds it).
- fix-consistency and fix-claims, LOW (merged) — card 12's "five pointers" count is wrong: fixed as proposed (the number is dropped).
- fix-consistency, LOW — `TestResolveGateSpec_ValidOneEntryPerVocabularyName`'s `parts` row needs a `Setting`: fixed differently.
  The row's env sets a `Setting` answering 6 for `tapestry.parts_max`, rather than `envkit.TemplateSetting`, because `envkit` builds `shedrecipe.Env` and a `shedrecipe` test importing it risks an import cycle.
- fix-claims, MEDIUM — card 18 claimed refusal-spec's no-reaching-test exemption for a refusal that has a way forward: fixed differently.
  The claim is removed, and a reaching test row is added through a CLI built with the darn recipe, since validate tests run the command without arming it.

Every fork returned a report.
The fix-consistency and fix-claims reports were saved after their fixes were applied, not before judging; their text is verbatim.

## Disputed

None.

## Deferred

None.
