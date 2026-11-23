```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 change set, looking for siblings or twins left stale.

1. **MEDIUM** — `contracts/specs/shed-recipe-spec.md:117-127`, the `escalate_decisions` and `blocking_escalates` paragraphs. The F1 fix added an exception, but the spec, which is the contract for both keys, was not updated.
   - Spec text: "When true, a judged round whose review holds a `decision`-class finding escalates with cause `decision`." It then says the arbiter settles it.
   - New behaviour in `escalateUndecided`: with `blocking_escalates` and a spent budget or hard cap, an open BLOCKING ledger entry of a class other than `decision` sends such a round to the parent with cause `budget`.
   - The `blocking_escalates` paragraph says nothing about decision findings or the arbiter carve-out.
   - Both the `escalate_decisions` and `blocking_escalates` paragraphs should state the exception. The `landing/summary.md` item 1 already does.

2. **LOW** — `contracts/stencils/bouncer/bouncer-template-escalation.md:18`, the `decision` cause description, is not wrong, but the `budget` bullet on lines 14-15 gives no sign that a round holding decision findings can now arrive as `budget`. An arbiter or parent reading a `budget` brief will see the detail naming decision findings (`; the review also holds decision findings (...), which the parent rules on`), so this is cosmetic. Also, the `budget` cause says "decide nothing", which matches the intent.

No other stale twins found:
- The `origin: focus` wording agrees across `roundcheck.go:70-73`, `:117-118` and `:329`, `burlerengine/doc.go:308`, and `burler-review-format.md:59,77-79`.
- `burler-step-review.md:57` agrees with them.
- `harden-rubric-review.md:54,105` still matches, since it only describes raising the finding.
- `docs/overview.md:1517` and `config.go:520` agree with each other.
- `template.yaml:53` was already updated.
- The reinterpretation ID `loom-code-auto-accept-escalates-blocking` matches between `reinterpret.go` and `configreg_test.go`.

Calls made:
1. `grep` for `origin: focus|FocusOrigin|origin: webster` across contracts, docs and internal, to find twin statements of the focus exemption that the fix did not touch. My context showed only the edited spans.
2. `grep` for `escalate_decisions|blocking_escalates|cause decision` in specs, stencils and docs, to find sibling descriptions of the changed routing.
3. `sed` of `shed-recipe-spec.md` lines 115-130 and `grep` of `bouncer-template-escalation.md`, to read the two key paragraphs and the cause descriptions. Neither was in my context.
