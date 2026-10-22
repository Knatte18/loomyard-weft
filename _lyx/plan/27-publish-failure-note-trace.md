# Card 27 — the Publish failure note names the traced cards

**Edit:**
- `internal/loomshed#checkedPublishFailure`
- `internal/loomshed#PublishFailureNote` -> `func PublishFailureNote(worktreeRoot, verifyDir, planDir string, tiers func(gate string) ([]int, error)) string`
- `internal/loomcli#loomCLI.wire`
- `internal/loomshed#TestPublishFailureNote`
- `internal/loomshed#TestPublishFailure`

**Create:**
- `plan:internal/loomshed#tracedLocation` -> `var tracedLocation *regexp.Regexp`
- `plan:internal/loomshed#tracedGlyph` -> `var tracedGlyph *regexp.Regexp`

**Intent:**
D9: the note the Webster-Review reviewer and fixer read after a Merge-In or Publish failure names the plan cards the trace matched.

`checkedPublishFailure` shape-checks the new fields as it does the others, dropping and logging each element that fails: a traced card needs a positive number, every glyph matching `tracedGlyph` (glyph characters only, no whitespace or backtick) and every location matching `tracedLocation` (a relative path of safe characters, a colon and a line number); a commit range keeps both ends only when each is a full hex object name (`fullObjectName`), else it is emptied; an untraced entry must match `tracedLocation`.
So no free text from the record reaches the prompt.

`PublishFailureNote` takes the plan directory and, after the failing tests, renders:
each traced card with its number, and its slug and first `Intent` line read from the live plan at render time (`planparser.ParsePlan(planDir)`), never from the record, its matched glyphs and locations, and its batch's commit range as the range to diff when starting the fix (`git diff <from> <to>`), or that Webster holds no range for it;
then the later range, as the later commits touching the same members;
then the untraced locations.
A card number the live plan no longer holds renders its number alone; a plan that does not parse renders the numbers alone, logged at Warn, so the note never fails.
An older record without the fields renders exactly as today.
`loomCLI.wire`, its one caller, passes `planparser.PlanDir(anchorPath)`.

`TestPublishFailure` gains rows over `checkedPublishFailure`: a card with a glyph holding a backtick, a location with a space and a range with a short name are each dropped, while valid ones are kept.
`TestPublishFailureNote` gains rows, writing a plan under a temp plan directory: a record whose card is in the plan shows its number, slug, first intent line, glyphs, locations and batch range; a record with a later range shows it; untraced locations show; a card number missing from the plan shows the number alone; an older record without the fields renders as before.
Its existing rows pass an empty plan directory and keep their expected text.

**ImpactSummary:** `PublishFailureNote` has one production caller, the `PublishFailure` seam in `loomCLI.wire`, re-wired here; `checkedPublishFailure` also feeds the verify gate's confirm steps, which read only the kept tests and tier and are unchanged.
