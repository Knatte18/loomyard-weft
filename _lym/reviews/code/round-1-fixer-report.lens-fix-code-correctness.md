```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (lens fix-code-correctness).

**`mergeDangerSection` (`internal/summaryparser/description.go`)**
- Traced input `# T\n\nb\n\n## Merge danger\n\n~~~sh\n# revert it\ngit revert HEAD\n~~~\n\n**Door:** two-way\n**Blast radius:** docs\n`. The fence lines and the fenced lines are skipped, the heading flips `found`, and the two labelled lines land in `section`. The result is nil findings, as the new row wants.
- Traced a fenced-only heading. `inFence` is true while the heading line is seen, so `found` stays false and the result is `no-merge-danger`.
- Traced a body that ends inside the section. The loop ends and returns `section, found`. Correct.
- The heading line itself is not appended, and `isSectionHeading` only runs once `found` is true, so a `# Title` before the section does not matter.

**Added test rows**
- Fenced-only section: the old code found the fenced heading and its valid labels, giving nil. The row wants `no-merge-danger`, so it fails with the defect in place.
- Fenced example before a real section with door `revolving`: the old code validated the example and gave nil. The row wants `merge-danger-door`, so it fails with the defect in place.
- Fenced `# revert it` comment inside the real section: the old code ended the section early and reported missing labels. The row wants nil, so it fails with the defect in place.
- Level-one heading ending the section: removing the `# ` clause lets the labels through, giving nil where the row wants two findings.
- Period strip: with the `.` strip removed, `` `One-way`. `` trims to ``One-way` ``, which is neither door value, so the row fails.
- Labels directly under the heading: it passes, and the one-line-late mutants named in F4 no longer exist after the rewrite. The row is harmless.
- `TestRecordBeamDescription` unedited-draft row (`beamDraft("Ship the beam", "...", nil)`): the draft carries door `one-way or two-way` and radius `one word` (two words). Both findings are raised, no commit is made, and the error text carries both check names. It would pass if the placeholders validated, which is what F7 asks it to catch.
- `chairWants` glossary assertion: it fails if `heddleengine.Glossary()` is swapped for other text. The gate was green.
- `levelOnly` row: `## lym's glossary` has the prefix `## `, and the other rows stay exact. No assertion was loosened.

**Findings**
- LOW — `isFenceLine` / `mergeDangerSection`: any ``` or ~~~ line toggles the fence, whatever its kind or length. A `~~~` block containing a ``` line, or a block that is never closed, mis-tracks the state. An unclosed fence before the real section reports `no-merge-danger`. This only affects unusual markdown and the review did not ask for it, so it is low.
- LOW — `contracts/stencils/loom/loom-template-discussion.md`: `{{.glossary}}\n## Step 5` now depends on `Glossary()` ending in a newline. If `glossary.md` ever lost its final newline, Step 5's heading would join the glossary's last line. The existing `\n\n## Step 5` assertion at `discussiontable_test.go:243` catches that, so it is not silent.

No callers of a changed signature are affected: `mergeDangerSection` still returns `([]string, bool)`, and `isFenceLine` is new and unexported.

Further tool calls made: none. My context already held every changed file.
