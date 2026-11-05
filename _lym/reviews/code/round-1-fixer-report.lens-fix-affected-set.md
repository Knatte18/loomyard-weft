```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets the fix had to cover (F3's stale-prose set, F1's notice sites).

Findings:

- LOW — `internal/shedrecipe/entries_bouncer.go:30`: the `bouncerEntry` doc says `escalate_decisions` makes a decision-class round "escalate to the run's parent". F3's fix reworded the same claim in `shed-recipe-spec.md:61`, but this twin keeps the old routing.
- LOW — `internal/shedadapters/bouncer.go:158`: the `EscalateDecisions` field comment says the round escalates "to the run's parent with cause decision". Its twin at `escalation.go:18` (the `EscalationCause` comment) was already changed, and `shedadapters/doc.go:108` was reworded by this fix.

Not findings:

- F1's set holds. `noticeArbiterSettled` has one caller (`drive.go:218`), and the driver doc, the `arbitrate` comment and the `goDriver` comment all state the new timing.
- `bouncerdecision.go:25` and `arbiterstop` say the decision halt is the parent's alone. That is the post-arbiter halt, so the text is still true.
- `loom-rubric-discussion-review.md:66` and `loom-template-discussion.md:107` define the finding class for Discussion. The review rejected the rubric line, and the template line was not named.

Calls made:
1. Grep for parent-only phrasings across Go, md and yaml, to find stale F3 twins beyond the sites the review named.
2. Grep for `noticeArbiterSettled` and the outcome-line text, to cover F1's callers and docs.
3. Grep for `decision escalation`, `cause decision` and `EscalateDecisions` lines that mention parent or operator, to find routing prose the first grep missed.
