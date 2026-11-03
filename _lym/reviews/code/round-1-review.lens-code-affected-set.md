```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — for each change, find what the diff forces outside its lines.

**Per change, the set found**

1. `loom-discussion-spec.md`, new `Review:` bullet.
   - `internal/discussionparser/decisionids.go:70-79` finds `D<n>` as a whole word anywhere in an acceptance bullet, so the `Review:` prefix needs no parser change. Covered.
   - `internal/discussionparser/validate.go` checks only headings and the PATTERN changes section. Covered.
   - Nothing in the parser reads `Review:`, so the mark is convention only and no code enforces it. This matches the summary's claim.

2. `loom-template-discussion.md`, Testing item.
   - The Chair and Advisor stencils include or sit beside this one. No marker was added, so the marker tests in `markers_test.go` are unaffected. Covered.
   - `loom-rubric-discussion-review.md:40` mentions acceptance criteria only for deferred facts. It does not contradict the new rule. Covered.

3. `loom-rubric-plan-review.md`, Test economy bullet.
   - Related text that still says only "stencil" or "assertion on a stencil's static prose":
     - `contracts/specs/loom-plan-spec.md:27`
     - `contracts/stencils/loom/loom-block-plan-format.md:16`
     - `pattern/PATTERN-test-speed.md:155` ("No test pins a stencil's static prose")
   - The plan writer is told only that "a stencil's static prose is not behavior a test covers". The rubric now flags a planned test of the static prose of any spec, doc or other text. The writer's side stays narrower, so it can still plan a spec or doc prose test that the rubric then flags. The rubric's symmetry note ("whatever the writer's stencil says not to write, this rubric must not flag as missing") is not violated. This is the opposite direction: the reviewer flags what the writer was never told to avoid. This is a gap in the set.

4. `loom-rubric-code-review.md`, Wording criteria bullet.
   - The Test economy bullet in the same file (line 109) still says "a new assertion on a stencil's static prose". The plan rubric now says "stencil, spec, doc or other text". The two rubrics are inconsistent in scope.
   - `rubric_test.go` checks the rubrics only for markers and range, so nothing asserts the new text. Covered.

5. The commit message and `_lym/landing/summary.md`.
   - Both match the diff. There is no `Pattern-Departure:` trailer, and no PATTERN entries were declared, so nothing is owed there. Covered.

**Findings**

- **F-A** (LOW, `consistency`): `contracts/stencils/loom/loom-rubric-code-review.md:109`, evidence also at `contracts/stencils/loom/loom-rubric-plan-review.md:59`. The code-review Test economy bullet "a new assertion on a stencil's static prose" was not widened to spec, doc or other text. The plan rubric was widened and the new Wording criteria bullet says a prose test never stands in. Fix: widen the bullet to "a new assertion on the static prose of a stencil, spec, doc or other text".

- **F-B** (LOW, `consistency`): `contracts/stencils/loom/loom-block-plan-format.md:16`, `contracts/specs/loom-plan-spec.md:27`, `pattern/PATTERN-test-speed.md:155`. These rule documents still limit the static-prose rule to stencils, while the plan rubric now cuts a planned test of any spec or doc prose. The plan writer is not told to plan none. Fix: widen the "stencil's static prose" wording in these lines to "the static prose of a stencil, spec, doc or other text", or accept the narrower writer-side text.

**Tool calls (2)**
1. `grep` for "acceptance" in the loom stencils, plus `sed` of `loom-block-plan-format.md` lines 10-20. My context had no view of the other stencils that mention acceptance criteria or of the plan writer's format block.
2. None other; the parser and PATTERN text came from the manifest.
