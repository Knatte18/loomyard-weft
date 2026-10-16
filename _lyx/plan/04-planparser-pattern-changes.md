# Card 4 — planparser-pattern-changes

**Create:**
- `plan:internal/planparser#PatternChange` -> `type PatternChange struct`
- `plan:internal/planparser#patternChangesHeading` -> `const patternChangesHeading = "## PATTERN changes"`
- `plan:internal/planparser#parsePatternChanges` -> `func parsePatternChanges(section []string) []PatternChange`

**Edit:**
- `internal/planparser#Plan`
- `internal/planparser#extractPlanSections`
- `internal/planparser_test#TestParsePlan_PlanLevelSections`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#extractSection`

**Intent:**
D5.3's plan half: the plan overview's optional `## PATTERN changes` section, parsed by `internal/planparser`, the plan's sole parser under `PATTERN-sole-parsers`.

`PatternChange` holds `ID`, `Action` and `Text` strings.
`Plan` gains `PatternChanges []PatternChange`, nil when the overview has no such section or the section holds no bullet.
`extractPlanSections` fills it from `parsePatternChanges(extractSection(body, patternChangesHeading))`, and its doc and `sections.go`'s file comment name the fourth section.

`parsePatternChanges` reads each non-blank line of the section that is not a fence line.
A line of the shape `` - `PATTERN-<id>` — <action>: <text> `` or `` - `PATTERN-<id>` — removes `` yields an entry with its ID (without backticks), action and trimmed text (empty for `removes`).
Any other non-blank line yields an entry with an empty `ID`, an empty `Action` and the trimmed line as `Text`, so the plan gate's comparison with the decision record (card 11) flags it instead of the parser dropping it.
The parser checks no action name: the record's own gate checks the shape, and the plan gate checks the plan's entries equal the record's.

`loom-plan-spec.md`'s `## On-disk layout` lists `## PATTERN changes` among the optional plan-level body sections, and a short paragraph there says it carries the decision record's `## PATTERN changes` entries, one bullet each, in the bullet shape `loom-discussion-spec.md` defines, without the record's added-after-Discussion suffix.

`TestParsePlan_PlanLevelSections` gains rows: an overview with a section of an `adds`, a `changes` and a `removes` bullet parses all three in order; a section with a line that fits no bullet shape yields the empty-ID entry carrying that line; a section whose bullets sit inside a fenced block yields the bullets and no entry for either fence line; an overview without the section leaves `PatternChanges` nil.

**ImpactSummary:** one new optional field filled at parse; no caller reads it until card 11, and every other section parses as before.
