# Card 4 — twill-host-values

**Edit:**
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective/doc.go`
- `contracts/stencils/twill/twill-directive.md`
- `internal/loomengine#twillBlocks`
- `internal/loomengine#TestDiscussionSpec_Twill`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#hostNeutralClaims`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/twilldirective#TestDirective_MissingValueNamesTheMarker` -> `func TestDirective_MissingValueNamesTheMarker(t *testing.T)`

**Intent:**
D4: the shared directive stencil gains the three host values Plan needs, and the reviewed files become one host-rendered phrase.
The Discussion host fills the new values with today's text, so its rendered prompt changes only on the reviewed-files line.

`contracts/stencils/twill/twill-directive.md`:
- The procedure's opening line ("When the design is settled and before you write any output file, run this procedure once:") becomes `{{.spawn_point}}` on its own line.
- Step 6's "Fix the record, or reject the finding with a one-line reason." becomes `{{.rejection_rule}}`.
- In the fork boilerplate's last line, only the parenthesised phrase `a severity (`BLOCKING`, `MEDIUM` or `LOW`)` becomes `{{.severity_rule}}`, so the line reads "Return your findings only as your final message, each with {{.severity_rule}}, the section or decision it concerns, a one-line summary and the evidence."
- "After `ready: true`, read the files under `{{.reviewed_files}}` and review them …" becomes "After `ready: true`, read {{.reviewed_files}} and review them through your lens as an outsider.", the marker outside backticks, since the host renders the phrase with its own backticks.
- The banner lists every marker by name without a count.
The stencil stays host-neutral.

`internal/twilldirective`:
- `Values.ReviewedFiles` changes from `[]string` to `string`, the phrase rendered verbatim into `reviewed_files`, and `Values` gains `SpawnPoint`, `RejectionRule` and `SeverityRule` strings, each documented as the text its marker renders.
- `Directive` fills `spawn_point`, `rejection_rule` and `severity_rule`, and `reviewed_files` with the phrase as given; every marker stays required, so an empty value or an empty `Lenses` fails the fill with `stencil.Fill`'s error naming the marker, wrapped with the stencil name.
- `doc.go`'s "What it says" names the spawn point, the rejection rule and the severity rule among the host values.

`internal/loomengine#twillBlocks` passes the Discussion host's values: `ReviewedFiles` is "`<decision record path>` and `<support log path>`", each path in backticks; `SpawnPoint` is "When the design is settled and before you write any output file, run this procedure once:"; `RejectionRule` is "Fix the record, or reject the finding with a one-line reason."; `SeverityRule` is "a severity (`BLOCKING`, `MEDIUM` or `LOW`)".

Tests:
- `TestDirective_RendersEveryValueVerbatim` passes a `ReviewedFiles` phrase and the three new values as sentinels, and asserts each lands, the reviewed phrase inside the line "read <phrase> and review them", and the severity sentinel between "each with " and ", the section or decision it concerns".
- New `TestDirective_MissingValueNamesTheMarker` is a table over a fully filled `Values` with one field emptied per row (`ReviewedFiles`, `SpawnPoint`, `RejectionRule`, `SeverityRule`, `Lenses` set to nil), asserting `Directive` errors and the error names that row's marker (`reviewed_files`, `spawn_point`, `rejection_rule`, `severity_rule`, `lens_list`); acceptance: "A missing host value or an empty lens list fails the directive render with an error naming the marker."
- `TestDiscussionSpec_Twill` additionally asserts the prompt holds "read `" + the decision record path + "` and `" + the support log path + "` and review them", today's spawn point line, today's rejection rule and "each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the section or decision it concerns", pinning that only the reviewed-files line changed.
- `TestStencils_TopLevelMarkers`'s `twill-directive` row adds `rejection_rule`, `severity_rule` and `spawn_point` in sorted order.
- `wordingClaims`' `twill-directive.md` entry claims `{{.spawn_point}}`, `{{.rejection_rule}}` and `{{.severity_rule}}`, and replaces nothing else; `hostNeutralClaims` also bars the Plan host's names `validate-plan`, `00-overview.md`, `lens-review.md` and `Card Index`.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/twilldirective` by one.

**ImpactSummary:** `Values` changes shape and `twillBlocks`, its one caller, follows; the Discussion twill prompt changes on the reviewed-files line only.

**Commit:** `4: Twill directive takes spawn point, rejection and severity rules from its host`
