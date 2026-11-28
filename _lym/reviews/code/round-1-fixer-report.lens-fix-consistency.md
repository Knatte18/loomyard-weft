```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1/F3 change to the two darn measurement-rule fragments.

**LOW** — the closed-scope stop report's way forward and the new settle clause disagree.
- **Changed text:** the new clause in `contracts/stencils/darn/darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`: "a `Decision for <slug>:` board note … that says the scope is closed and files the measurements for a new task settles the point".
- **Problem:** the unchanged way-forward sentence a few lines below, in both files, tells the stop report to name only "a board note filing the measurements for a new task". It gives neither the `Decision for <slug>:` title nor the requirement that the note say the scope is closed, and the new clause requires both. A note written exactly as the stop report asks does not satisfy the clause, so the resumed writer stops again.
- **Sibling wording:** `darn-template-write.md:62` asks for "a board note saying the scope is closed". `darnStopWayForward` (`internal/loomshed/darnwrite.go:32`) names both the title and the closed-scope statement. The halt Reason therefore still carries the right instruction, which is why this is LOW.

I found no other disagreement:
- The new clause matches Step 3 of the template (lines 53-55 and 60).
- The two fragments carry identical wording.
- The comments in `spec.go`, `stencils.go` and `spec_test.go` and the fragment headers still hold.

Tool calls:
1. One `grep` for other statements of the measurement rule across `.go` and `.md` files, because my context did not show whether other twins exist.
