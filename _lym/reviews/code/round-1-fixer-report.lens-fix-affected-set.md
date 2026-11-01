No findings from the fix-affected-set lens. The one site the fix did not touch that restates the sentence is out of scope.

- **F1:** the constant is built into the delivery text at `settings.go:450` and compared in `settings_test.go:803`, `prepare_test.go:398` and `prepare_test.go:520`. All four use the constant itself, so none carries a copy of the old wording. `doc.go:225` still describes the sentence as "rather than again later". The added exception makes that slightly incomplete, but the review said no `doc.go` edit was needed, and the fix did not make it worse.
- **F2:** `docs/overview.md:419` is the only description of the appended sentence in the docs, and the fix updated it. `docs/research/turn-audit-2026-10-28/report.md:228` quotes an older version of the sentence. It is a dated measurement report, which the prose skill exempts from staying current.
- **F3:** the comment at `settings.go:440` is the only doc comment on `batchingSentence`.

Findings left unchecked: none.

Calls made:
1. `grep -rIn` across the tree for "whose arguments", "batchingSentence" and "reading it again". The context had no full list of the places that restate or consume the sentence.
2. `sed -n 225p doc.go`. The earlier grep truncated that line.
