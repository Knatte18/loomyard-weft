---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D4
    summary: "D4 names the landing routes that skip the judged review but omits the auto-accept and hard-cap routes that accept a stencil finding unfixed, so its bound overstates"
    fix: "In D4's Rationale, extend the what-it-lets-through list with a loom run whose `webster_auto_accept` is true, where the Webster-Review perch accepts a harvested round at a spent budget instead of running the soft-cap rule, and with a round numbered at or past the hard cap (`webster_hard_max_bounces`, or `review_hard_max_bounces` for darn), which is accepted or escalated and never continued. Reword the bound sentence from passing a judged Burler round to the reviewer reading stencil changes against the stick and a finding being fixed or escalated unless the perch's auto-accept or hard cap accepts the round. Darn has no auto-accept key, so the darn clause names only the hard cap."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D1,D2,Acceptance criteria
    summary: "D1 keeps marker-presence assertions but D2 sends every other claims_test row away, so a marker removed from a stencil whose only marker test is a claims_test row fails no test"
    fix: "In D2, add a bullet before `every other row goes`: a claims_test row that names a marker without pinning its position, for a stencil that no Fill round-trip or marker-set test reads in its real file, becomes a row of `TestStencils_TopLevelMarkers` in `markers_test.go`, one row per stencil. The darn Describe stencil `landing-template-describe-darn.md` is such a stencil, since `internal/landingshed/describe_test.go` fills a fixture string and not the real file. Have the plan writer sort each marker-naming claims_test row by whether another test reads the real stencil's markers, and state that the acceptance criterion `Removing a marker from a stencil ... still fails a test` holds for every stencil the rows covered today."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D2
    summary: "D2's definition of a section-scoped marker row (a section key whose must holds a marker) also matches the Step 5 specs_dir row, whose must carries a prose path, against the rule that kept rows anchor on markers, headings and the end of the body only"
    fix: "In D2's marker-position bullet, narrow the definition to a `section:` row whose `must` is exactly one marker, and add that the `{{.specs_dir}}/loom/loom-discussion-spec.md` row under `## Step 5 — Write the two output files` goes, since its `/loom/loom-discussion-spec.md` tail is prose of one stencil. The rows that stay under the narrowed definition are the `{{.twill_directive}}`, `{{.decision_record_path}}`, `{{.support_log_path}}`, `{{.parent_notify}}` and `{{.arbiter_command}}` rows."
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D2,Acceptance criteria
    summary: "D2 says the kickstart_pack_inline marker precedes the sandbox heading and a section row checks the marker sits inside the section, so a move within those bounds passes while the criterion says moving a position-pinned marker still fails"
    fix: "In D2, state the kickstart_pack_inline row as the marker followed by a blank line and `## The sandbox rule` directly (`{{.kickstart_pack_inline}}\\n\\n## The sandbox rule`), which holds only a marker, whitespace and a heading. In the acceptance criterion, reword `moving a position-pinned marker` to `moving a position-pinned marker out of its anchored place (out of its heading's section, or off the end of the body or its heading)`."
---

### [MEDIUM] D4's bound omits the auto-accept and hard-cap routes

**Location:** `_lyx/discussion/decision-record.md`, D4 Rationale.
**Issue:** The focus asked whether D4 holds for a zero Webster-Review budget and a darn `review_max_bounces` of 0, and whether every landing route is stated with its bound.
Both zero-budget sentences are accurate.
`internal/loomrecipe/doc.go` says a resolved budget of 0 stamps `zero_rounds` on the Bouncer row, which settles approved at once, and the segment keeps its route.
`contracts/recipes/darn-recipe.yaml` and `internal/darnengine/template.yaml` say a darn budget of 0 routes Darn-Write straight to Merge-In and runs neither the review nor Describe.
The list is incomplete for budgets above 0.
`internal/loomengine/config.go` and `internal/loomengine/template.yaml` declare `webster_auto_accept`, which makes the perch accept a harvested round at a spent budget instead of running the soft-cap rule, so a stencil-regression finding is accepted unfixed.
`internal/darnengine/template.yaml` says a round numbered at or past `review_hard_max_bounces` is accepted or escalated and never continued, and the same hard-cap rule holds for loom.
D4's bound sentence says such a landing `passes a judged Burler round` and `reads stencil changes against the stick`, which holds for the read but not for the finding being acted on.
The attack-surface test asks the Decision to state what the downgrade lets through and what bounds it, and these two routes are lets-through items with no stated bound.

### [MEDIUM] Marker-presence rows have no home in D2

**Location:** `_lyx/discussion/decision-record.md`, D1 first keep, D2 `every other row goes`, Acceptance criteria.
**Issue:** D1 keeps `a marker set, a required-marker error, a Fill/FillOptional round-trip`.
D2 places three kinds of row and then drops `every other row`.
`claims_test.go` holds many `wantAll` rows that pin a marker with no position, such as `{{.card_read}}`, `{{.card_gates}}`, `{{.batch_gate}}` and `{{.decision_record_path}}` in the darn Describe row.
For the webster templates, `internal/websterengine/template_test.go` fills the real stencils (`TestTemplates_FillRequiresEveryRequiredMarker`), so dropping their rows loses nothing.
For `landing-template-describe-darn.md` the only other test is `internal/landingshed/describe_test.go`, which fills a fixture string (`darnStencilFixture`), so a marker deleted from the real stencil would fail no test once the row goes.
`markers_test.go`'s `TestStencils_TopLevelMarkers` covers a fixed subset of stencils and does not list the darn Describe stencil, the plan template or the discussion template.
The acceptance criterion `Removing a marker from a stencil ... still fails a test` is then false for those stencils, and the Goal's `tests keep pinning only what Go fills, reads or names` loses a Go-filled marker.

### [MEDIUM] The section-scoped marker definition catches a prose-bearing row

**Location:** `_lyx/discussion/decision-record.md`, D2 marker-position bullet.
**Issue:** D2 defines the rows it keeps as `a claims_test.go row with a section: key whose must holds a marker`.
`claims_test.go:321` is `{must: "{{.specs_dir}}/loom/loom-discussion-spec.md", section: discussionStepFiveSection}`, whose `must` holds a marker and a static path tail.
Read literally, D2 keeps it, and its check would anchor on the prose tail `/loom/loom-discussion-spec.md` and on the heading `## Step 5 — Write the two output files`, against D2's own `anchored on markers, headings and the end of the body only, never on prose`.
D1 would drop it as static prose of one stencil.
The two rules disagree on this row, and a plan writer sorting by the literal definition keeps a prose assertion.
Round 2's fixer report records that the definition was meant to cover the `{{.specs_dir}}` row at line 321 and disputed nothing, but it does not address that the row's `must` carries a prose tail, so the contradiction with the anchor rule stands.
The remaining section-scoped marker rows (`claims_test.go:280,324,325,329,1092,1093`) hold a bare marker and satisfy both.

### [LOW] Kept position rows are looser than the acceptance criterion says

**Location:** `_lyx/discussion/decision-record.md`, D2 and the fifth acceptance criterion.
**Issue:** D2 says the `{{.kickstart_pack_inline}}` row `checks the marker precedes ## The sandbox rule`, and a section row `checks the marker sits inside that heading's section`.
A marker moved to the top of the body still precedes the heading, and a marker moved inside its section still sits there, so neither move fails.
The criterion says `moving a position-pinned marker ... still fails a test`.
Today's row pins adjacency (`{{.kickstart_pack_inline}}\n\n## The sandbox rule`), which is already prose-free, so stating it keeps the existing strength at no cost.
The choice between adjacency and plain order is the author's, and the criterion should name whichever it picks.

## Rejected

- Focus item 2, whether D1's keep list and D2's marker-position bullet agree on a Go-filled marker inside a heading's section such as `{{.twill_directive}}` under a `## Step` heading: they agree.
  D1 keeps the position of a marker relative to a heading, D2 keeps the `{{.twill_directive}}` rows in the plan template's `## Step 2 — Explore the codebase` and the discussion template's `## Step 4 — How to get answers`, and moving the marker out of its section fails the kept row.
  The only gap is a move within the section, filed under the LOW finding.
- Focus item 3, whether the rewording criterion lists the headings a kept row anchors on, such as `## The sandbox rule`: the criterion names the category `a heading a kept marker-position row anchors on`, which covers `## The sandbox rule`, `## Step 2 — Explore the codebase`, `## Step 4 — How to get answers`, `## What you may write`, `## Notifying the parent` and `## Arbiter` without enumerating them.
  Enumerating them would pin a perishable list, so the generic phrase is the right form.
- Focus item 3, the `{{.kickstart_pack}}` row anchored on the end of the body: D2 states it as `checks the body ends with the marker`, which holds only a marker and the body end, and rewording the sentence before it fails nothing.
- D4 claims for the zero-budget routes and the `//lyx:guard` mechanism named in D2: both match the code and the repo's `PATTERN-test-speed` text.
