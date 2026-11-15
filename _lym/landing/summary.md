# Darn writer asks its parent to run a reported failure's test first

A darn task that starts from a reported test failure now checks whether the failure still reproduces before it diagnoses it.
The darn prompt forbids `lym gate test` for a writer whose verify gate has a cold fixer,
so the writer used to trace the failure by hand, even when the failure was already gone on its base.
Issue #119 reported a long manual trace of a failure that one test run showed no longer existed.

The gated Step 4 list in `contracts/stencils/darn/darn-fragment-make-change-gated.md` now opens with an ask to the parent.
When the task starts from a reported test failure, the writer sends `lym creel post --parent --expect-reply`
before it diagnoses or edits, and ends its turn to wait.
The message names the test exactly and its tier, and asks the parent to run it once on the writer's base through `lym gate test`.

- When the parent reports that the run's report shows the named test ran and passed,
  the failure no longer reproduces: the entry is one that landed work overtook, and Step 3 settles it.
- When the report does not show the test ran, because it was skipped for another tier or matched no test,
  the writer diagnoses the failure as usual.

The doc comments in `contracts/stencils/stencils.go` and `internal/darnengine` (`doc.go`, `spec.go`) describe the new ask.
No Go logic changes.

Closes #119

## Design decisions

- The writer asks its parent for the run instead of being allowed to run `lym gate test` itself.
  This keeps the no-gate rule intact for the writer and its subagents, and the parent decides the single run.
- A passing result counts only when the report shows the named test ran.
  A skipped or unmatched test must not read as a pass, so that case falls back to ordinary diagnosis.
- The change is confined to the gated fragment; the ungated variant already runs the gate after each commit.

## To check by hand

Nothing was run by hand, and no regression check ran for this task.
Check these findings, which a review left open:

- The gated fragment's ask names the test and its tier; check it does not also pin a scope form or `go test` flags
  that the parent's `lym gate test` call should choose (finding `gated-fragment-names-scope-form-and-go-flags`).
- A run that skips the test or matches none must not read as a pass; check the fragment's wording
  makes that fallback unambiguous (finding `reproduce-run-can-skip-test-and-read-as-pass`).
- A spec test may pin the gated stencil's prose; check that the test and the fragment's new text agree
  (finding `spec-test-pins-gated-stencil-prose`).
