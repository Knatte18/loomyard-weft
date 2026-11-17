---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/lenses/template.yaml:310-313
    summary: With a decision record alone as the answer key, code-scope traces changes to the writer's own D<n> headings, which the rubric forbids as a measuring stick, so a writer can license its own unrequested scope.
    fix: >-
      In the `code-scope` lens of internal/lenses/template.yaml, replace the sentence
      "List the record's decisions: each `### D<n> — <title>` heading of its `## Decisions` section
      and each of its `Added after Discussion` entries; a record with no decision id lists each goal
      point of the amended board entry instead." with
      "List the decisions: where a plan is the answer key, each `### D<n> — <title>` heading of the
      record's `## Decisions` section; where a decision record alone is, each goal point of the
      board entry under its `## Goal`, never the record's other sections; and in both cases each of
      the record's `Added after Discussion` entries.", keeping the YAML block's indentation.
      Then rewrite the second and third sentences of _lym/landing/summary.md's opening paragraph to
      describe the same split: D<n> headings with a plan, the board entry's goal points with a
      decision record alone, and the Added after Discussion entries in both.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: docs/dotnet-hub.md:202-210
    summary: The documented hub review-code fan omits code-scope, yet the doc says the entry repeats the template's lenses, so a hub copying it silently drops the new lens.
    fix: >-
      In docs/dotnet-hub.md's "The repo-rules lens" section, add `    - code-scope` to the
      `review-code` fan of the YAML example, after `    - code-tests` and before `    - models-rules`,
      so the example again repeats the template's whole review-code list and appends models-rules.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/lenses/config_test.go:441-444
    summary: The new code-scope row in the lens-phrase table asserts the static prose of a lens, which the rubric's test-economy rule forbids for a new assertion.
    fix: >-
      Delete the `"code-scope": {...}` entry, its two sentence strings and closing brace, from the
      lens-phrase map in TestConfigTemplate_DecodesAndIsSelfConsistent in internal/lenses/config_test.go.
      Then delete the sentence "It also pins the lens's two finding sentences." from the `## Tests`
      section of _lym/landing/summary.md.
---

### [MEDIUM] code-scope takes writer-claimed decisions as the measuring stick in a record-alone run

**Location:** internal/lenses/template.yaml:310-313

**Issue:** The lens lists each `### D<n> — <title>` heading of the record's `## Decisions` and falls back to the board entry's goal points only when the record has no decision id.
In a darn run the answer key is the decision record alone, and the code-review rubric (`contracts/stencils/loom/loom-rubric-code-review.md`, "A decision record alone.") says the record's sections besides `## Goal` and its `Added after Discussion` entries are the writer's own claim, never the measuring stick.
A darn writer who fills `## Decisions` with `### D2 — also refactor X` makes every change toward X trace to D2, so the lens reports no unrequested scope and the board entry's first goal point goes unenforced.
`Record-Write` writes `## Decisions` empty (`internal/loomshed/recordwrite.go`, `recordSections`), so only a writer's own edit fills it, which is the case the rubric guards against.
Two forks and the handler found this independently.

### [MEDIUM] dotnet-hub doc's review-code fan example goes stale

**Location:** docs/dotnet-hub.md:202-210

**Issue:** The doc shows a hub `review-code` fan of `code-correctness, code-affected-set, code-tests, models-rules` and says a fan entry replaces the template's whole list, "so the entry above repeats the template's lenses and appends the new one".
After this diff the template fan also holds `code-scope` (`internal/lenses/template.yaml:423`), so that sentence is false, and a hub following the guide runs code review without the new lens.
The diff's own lens names a doc a traced change makes stale as part of the change, and this one is left out.

### [LOW] New lens-phrase row asserts static prose

**Location:** internal/lenses/config_test.go:441-444

**Issue:** The rubric's test-economy bullet makes "a new assertion on the static prose of a stencil, spec, doc or other text" a finding.
The added `code-scope` row asserts two sentences of the lens's prose.
Its count, order, procedure-lens shape, csharp superset and emptied-fan behaviour are already pinned elsewhere in the same test and in `TestCheckLoad`, so dropping the row loses no behaviour coverage.
The table's older rows predate this diff and are not this round's subject.

## Focus directives

- Both halves of the board entry are carried: "A change that traces to no decision is unrequested scope" and "A decision no change builds is a finding of class `scope` too." Neither is weakened. The forced-edit and `Pattern-Departure:` exemptions narrow the first half to what the rubric's own scope bullet already exempts.
- Decision-id source: `### D<n> — <title>` matches `decisionHeadingPrefix` and `decisionHeadingSeparator` in `internal/discussionparser/decisionids.go`, and `Added after Discussion` entries match `isAddedDecisionHeading`. The fallback's condition is the defect in F1.
- csharp layer: `code-scope` sits after the three base entries and before `numerics`, so the base fan is a prefix in base order. go and python layers define no `review-code` fan. Template and csharp agree.
- config_test.go: `wantLenses = 38`, `"review-code": 4`, the four-entry order and the four-name `remove:` list all match the template. The phrase row is F3.
- Scope: the diff adds only the lens, its two fan wirings and the test updates. It misses the stale doc in F2. No PATTERN entry names the review-code fan's members, and the directive declares no PATTERN change.

## Rejected

- Commit message and summary claim "the csharp layer's review-code fan keeps the base entries first": the diff bears it out; no finding.
- Summary claim that a hub-tuned review-code fan keeps its list as written: `internal/lenses/doc.go` Reconcile section confirms it; no finding.
