```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's three prose changes force outside the lines they touch.

**Per change: the affected set I found, and whether it is covered**

- **Spec `### A bug entry's record`** (`contracts/specs/loom-discussion-spec.md:135-146`). It sits under `## The decision record`, which covers every loom record.
  - Discussion-Write records: covered, because the stencil now asks for all three items.
  - Darn `Record-Write` records: **not covered** (F1). A stored Sketch read through `ReadStored` (`internal/loomshed/recordwrite.go:26,177`) and Discussion-Review are covered.
  - `discussionparser` and the discussion gates: covered. The rule is review-only, so no parser change is needed (`docs/overview.md:1041-1047`).
  - Red-first gate behaviour: covered.
    - The waiver comes only from the parent's added entry, matched by case-insensitive prefix (`internal/loomshed/redfirstgate.go:57-60`).
    - The default base is the merge base with the parent branch (`internal/gatecli/redfirst.go:63,95`).
    - "Reaches the bug through code the merge base already has" matches `contracts/stencils/darn/darn-template-write.md:108` and `redfirstgate.go` (the `RedFirstGreenExit` way forward).
  - `docs/overview.md:1108-1109`: consistent.
  - `internal/shedrecipe/doc.go:204-213`: consistent.
  - `contracts/stencils/rubric_test.go` and `markers_test.go`: no assertion reads bug wording, so no fixture breaks.
- **Stencil paragraph** (`contracts/stencils/loom/loom-template-discussion.md:136-140`).
  - Header comment, no `{{if}}` rule: covered, since the paragraph adds no conditional and no marker.
  - Markers test: covered, nothing new.
  - Engine-terms scan: covered, no engine term.
  - No-denied-recovery scan: covered, no denied form.
  - Labels: Step 1's `lym board get` envelope carries `labels` (`internal/boardengine/view.go:53`, `store.go:47`). Covered.
  - The chair stencil includes this file whole, so the paragraph reaches it. Covered.
  - PATTERN-stencil-ownership "Producer pointers": the paragraph names the three items the board entry asks the stencil to ask for and points at the spec for placement. It does not restate the format. Covered.
- **Rubric bullet** (`contracts/stencils/loom/loom-rubric-discussion-review.md:451-455`).
  - Symmetry note (`:462-464`): the writer stencil now asks for the same three items, so the rubric flags nothing the writer was told not to gather. Covered.
  - Bouncer and burler consumers interpolate the rubric whole, with no new marker. Covered.
  - `rubric_test.go`: no assertion on this bullet. Covered.
- **Commit 084ae61f message**: every claim (spec in Goal/Constraints/AC, stencil settles all three, rubric flags an omission) matches the diff. It has no `Pattern-Departure:` trailer, and none is needed. Covered.
- **Declared PATTERN changes**: none declared. Nothing to land.

**Findings**

- **F1** — MEDIUM, `consistency`, `contracts/specs/loom-discussion-spec.md:137`.
  - The new rule is not scoped to Discussion-Write records. It says "A record for a board entry labelled `bug` states up front…", inside the section that governs every loom decision record.
  - Darn's `Record-Write` writes the record for exactly the runs the red-first gate guards: the gate is on darn's `Darn-Write` only (`contracts/recipes/darn-recipe.yaml:135-140`). That record holds the board body under `## Goal` and leaves `## Constraints` and `## Acceptance criteria` empty (`internal/loomshed/recordwrite.go:46-64`).
  - So every bug entry's Record-Write record breaks the rule as now written.
  - The fasit scopes the change to "When Discussion-Write runs for an entry labelled `bug`" (`_lym/discussion/decision-record.md:9`).
  - Fix: scope the rule to a record the Discussion-Write lead writes, the way the citations subsection names its row.

Tool calls:
1. Grep of `rubric_test.go`, `markers_test.go`, `docs/` and the other contracts for bug, red-first and spec references. The manifest did not cover the tests and docs that might assert or describe the changed prose.
2. Read of `docs/overview.md:1036-1060`, `internal/loomshed/recordwrite.go` section headings, and the overview's Discussion-Review lines. This checked whether Record-Write's records, which the new spec rule reaches, satisfy it; the manifest held only `ReadStored`.
