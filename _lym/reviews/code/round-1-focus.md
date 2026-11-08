---
round: 1
focus:
  - "internal/impactset/mains.go: does main-package discovery cover every main package of the module, and what happens when the module is empty, `go list` fails, or the tree has nested modules?"
  - "internal/gatecli/scope.go and cli.go: does `--main-packages` apply only to a `--card` scope under the go scoper, and do other scopers add nothing, as the Sketch decides?"
  - "internal/websterengine/cardgate.go, beginbatch.go, recoverbatch.go: is `--main-packages` passed for the last batch only, and does the recovery path agree with the begin path on which batch counts as last?"
  - "Scope against the decision record: the diff must add no `gates.batch.last_packages` field, no config step and no hard-coded Go path in templates, and must leave the plan-level verify untouched."
  - "contracts/stencils/webster/webster-body-implementer.md and govalues_test.go: does the stencil change keep every existing rule and match what the gate now does?"
  - "The change description, when present: does it state the measured choice between every batch and the last batch, with its numbers, as the Goal requires, and does the diff bear that out?"
---

Seed focus from the diff stat and the decision record.
