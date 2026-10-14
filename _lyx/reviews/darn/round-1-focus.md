---
round: 1
focus:
  - "contracts/stencils/twill/twill-directive.md: does the reordered procedure read coherently from both templates, given the directive sits between Step 4 and Step 5 in Discussion and between Step 2 and Step 3 in Plan while the spawn point points forward to later steps?"
  - "Relaunch rules in the directive: does a writer that died before its Agent call spawn exactly once after writing and a passing self-check, and does a writer that died at or after its Agent call never spawn again?"
  - "Ready-marker removal: grep for leftover references to the marker, `ReadyMarker`, `WriteScope`, `twill-write-scope`, `PrepareDiscussionSpawn` and `PreparePlanSpawn` in code, stencils, contracts, docs and tests; does anything still read what was removed?"
  - "internal/loomengine/twill.go and the DiscussionWrite and PlanWrite preparation: are the fresh-spawn preparations as the summary claims (parent-review `PrepareRound` alone, plan-directory rotator alone), and does the blank marker line leave single and seats prompts byte-identical?"
  - "Docs in the same diff: do `internal/twilldirective` doc, the loomengine file docs and `docs/overview.md` match the new behavior, and does PATTERN or the test-budget file need an edit the diff omits?"
  - "Test economy: do removed and changed tests leave the new forks-after behavior covered, and does any new test duplicate an existing one?"
---

Seed focus drawn from the board entry's scope list and the summary's claims, to be checked against the diff.
