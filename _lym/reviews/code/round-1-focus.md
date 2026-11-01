---
round: 1
focus:
  - "internal/gatecli/failurereport.go: does the report list every failure class the board entry names (build errors, test failures, vet and guard failures, the gate key's lint findings) with package, test, file:line and expected/actual or first panic lines? Check for any class or run path (tier, card, capped, detached) that closes without it."
  - "internal/gatecli/failurereport.go and counters.go: is the report bounded so it stays one tool result, and does truncation drop failures silently or say how many were cut? Check the full-log path and the touched-files list are always printed."
  - "internal/gatecli/run.go, tierrun.go, detachfront.go: do gate lint steps run on every path that runs tests, do their findings reach the report, and does a lint failure change the exit code and the done answer correctly? Check the interaction with the prompt-cache-lifetime answer from the previous commit."
  - "internal/gateslot/config.go, tiers.go, template.yaml and their tests: is the new lint-step key validated (unknown or empty steps, existing configs without it) and does the template's card key run the commands the Webster stencil now names?"
  - "The capped-call change (no stdout replay past 4096 bytes): is the limit applied consistently in code, doc.go and tests, and does a failing capped call still surface its failures?"
  - "contracts/stencils (loom-rubric-code-review.md, webster-body-implementer.md) and contracts/specs/loom-plan-spec.md: read against the board entry, does any stencil rule get dropped or weakened, and do the docs the change touches (docs/overview.md, doc.go files, pattern/PATTERN-gate-slot.md) match the code?"
  - "internal/websterengine and internal/loomshed edits: are they required by the new gate behavior or unlicensed scope?"
  - "Test economy in failurereport_test.go, counters_test.go and test_integration_test.go: any test duplicating another's coverage, a per-helper test the public report test already covers, or new behavior left uncovered?"
---

Seed focus from the diff stat and the board entry (the decision record holds only `## Goal`).
The commit carries no `Pattern-Departure:` trailer.
