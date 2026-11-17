```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: tracing what the `code-scope` lens and the `review-code` fan change force outside the diff's lines.

**Per change: the set found, covered or not**

1. **New `code-scope` lens prose** (`internal/lenses/template.yaml:309-323`)
   - It cites the rubric bullet "Scope against the answer key". That bullet exists at `contracts/stencils/loom/loom-rubric-code-review.md:65`: covered.
   - The decision-id source is the `### D&lt;n&gt; — &lt;title&gt;` heading of `## Decisions` (`internal/discussionparser/decisionids.go:16-19`, `parseDecisionHeading`), plus the `### Added after Discussion (…)` entries (`internal/discussionparser/identity.go:13`, `isAddedDecisionHeading`). Both match: covered.
   - The rubric's decision-record-alone rule (`contracts/stencils/loom/loom-rubric-code-review.md`, rendered in the round prompt's Rubric section) contradicts the lens. Not covered (finding 2).
   - The `procedureLenses` check in `config_test.go` puts `code-scope` in the review- fan set. The lens ends on "Your list: …" with no emphasis line, so it is consistent: covered.
   - No PATTERN entry or `pattern/` background file describes lens prose or the `review-code` fan's members. `grep lens PATTERN.md pattern/` returns only `lensreport`, leaf-package and config-layers lines, none naming the fan's lenses: nothing to cover.

2. **`review-code` fan gains `code-scope`** (`internal/lenses/template.yaml:423`)
   - `internal/lenses/config_test.go` covers the fan counts, the order and the emptied-fan `remove` list.
   - `docs/overview.md:1599` names the fan only, not its members: still true.
   - `internal/loomengine/template.yaml:19` and other `code_reviewer_fan` users name only the fan: unaffected.
   - The docs' copy of the fan list in `docs/dotnet-hub.md:202-210` is not covered (finding 1).
   - `MaxFanLenses = 16` (`internal/lenses/config.go:47`): the new size is 4 in the base and 5 in csharp, within the limit.

3. **csharp layer `review-code` fan** (`internal/lenses/sets/csharp.yaml:49-54`)
   - The base entries come first in base order with `numerics` appended, per `internal/lenses/doc.go` ("A layer fan holds the base fan's entries first, in base order"). The superset check in `TestConfigTemplate_DecodesAndIsSelfConsistent` covers this: covered.
   - `go.yaml` and `python.yaml` define no `review-code` fan (the grep found none): nothing to update.

4. **Commit messages and departures**
   - `edb4430b` carries no `Pattern-Departure:` trailer, and the diff departs from no PATTERN entry: no finding.

5. **Declared PATTERN changes**
   - The directive declares none: no finding.

**Findings**

1. **MEDIUM, consistency.** `docs/dotnet-hub.md:202-210` (and the sentence at 209-210) is now stale.
   - The documented hub `review-code` fan lists `code-correctness, code-affected-set, code-tests, models-rules`.
   - The doc says "A fan entry replaces the template's whole list, so the entry above repeats the template's lenses and appends the new one."
   - The template fan now also holds `code-scope` (`internal/lenses/template.yaml:420-423`), so the doc's claim is false.
   - A hub that copies the example silently drops the new lens.
   - The diff makes this doc stale and does not update it.
   - Evidence: `docs/dotnet-hub.md:203-207`, `internal/lenses/template.yaml:423`.

2. **MEDIUM, design.** `internal/lenses/template.yaml:310-313`, the `code-scope` decision source, contradicts the code-review rubric when the answer key is a decision record alone.
   - The lens always lists each `### D&lt;n&gt;` heading of the record's `## Decisions` section as a decision.
   - The rubric's record-alone form says the record's sections other than `## Goal` and its `Added after Discussion` entries "are the writer's own claim … never use them as the measuring stick" (`contracts/stencils/loom/loom-rubric-code-review.md`, the "A decision record alone" bullet, rendered in this round's Rubric).
   - In a darn run whose writer filled `## Decisions` with its own D-headings, any change traces to a decision the writer licensed itself, so the lens cannot flag unrequested scope.
   - The goal-point fallback applies only when the record has no ids. It should apply whenever the answer key is a decision record alone, alongside the `Added after Discussion` entries.
   - Evidence: `internal/lenses/template.yaml:310-313`, and the rubric's "A decision record alone." bullet.

**Tool calls**

1. A grep of docs, stencils and PATTERN for the `review-code` lens names: my context lacked the PATTERN and `pattern/` lens mentions and any other doc listing the fan's members.
2. A grep of the rubric stencil's "Scope against the answer key" bullet and the code-review lens docs: my context lacked the stencil source's text, which I needed to confirm the lens's cross-reference.
