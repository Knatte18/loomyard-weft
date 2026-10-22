---
verdict: APPROVED
---

Review range: `git diff de6b8465f..HEAD` (merge base with `main`, from `product.parent`), one commit touching three stencil files.

## Scope against the board entry

The entry asks for three changes, and the diff makes exactly those three:

- The Discussion writer researches the facts the design rests on (`loom-template-discussion.md`, Step 2).
- The Discussion reviewer flags a deferred research question or a halt-on-discovery rule (`loom-rubric-discussion-review.md`).
- The Plan reviewer flags a card carrying a research step (`loom-rubric-plan-review.md`).

The diff makes no other change; the decision record has no `Added after Discussion` entries.

## Stencil wording

- The Step 2 fence (no exact signatures, `file:line` citations, interface shapes, dependency lists, exhaustive existing-pattern research) is untouched, and the new paragraph states that it "excludes plan-level detail, never a fact the design rests on".
- The Discussion-Review bullet's last line, "A fact the writer was told not to gather is no such question", keeps the symmetry note in the same file holding.
- The Plan-Review bullet's fix route ("in the decision record through the parent") matches the neighbouring answer-key bullet.
- The new text adds no `{{` marker or conditional, so the writer stencil's leading-comment rule and the rubric marker tests are unaffected.

## Change description

The claims in `_lyx/landing/summary.md` hold against the diff: the chair stencil includes the template whole (`loom-template-discussion-chair.md:7`), the writer, advisor and darn stencils are unchanged, and neither rubric gains a marker.

## Repository rules

The added lines name no PATTERN entry, `pattern/` file or `lyx` command, one sentence sits on each line, and the change adds no module, CLI behaviour or cross-cutting infrastructure, so no `doc.go`, `docs/overview.md` or PATTERN edit is owed.
The diff adds no test, which `PATTERN-test-economy` requires for stencil prose.

## Rejected

- Fork lenses code-correctness, code-affected-set, code-tests and code-pattern-fit each returned no findings.
- Handler: the Discussion-Review exemption ("a fact the writer was told not to gather") could cover a fenced-class fact that a design does rest on, which the writer rule still tells the writer to research. Rejected as a finding: the overlap is a narrow wording edge, and the rubric's symmetry note requires the exemption to follow the writer's fence.
